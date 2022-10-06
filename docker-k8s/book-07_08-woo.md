## 설정 데이터 공유를 위한 리팩토링

사용하는 값들을 변수화시켜서 사용할 수 있다.

```
# 값들을 변수화한다.
variable app_name {
    default = "flixtube"
}
variable location {
  default = "West US"
}
```

```
# before
resource "azurerm_resource_group" "flixtube" {
  name     = "flixtube"
  location = "West US"
}

# after
resource "azurerm_resource_group" "flixtube" {
  name     = var.app_name
  location = var.location
}
```

## 쿠버네티스 클러스터 만들기

terraform registry의 azure kubernetes 항목을 보면 `identity` 또는 `service_principal` 항목이 필수이다. 인증정보를 위해 azure subscription id가 필요하고 이 id로 서비스 주체를 생성한다.

```
$ az account show

{
  "environmentName": "AzureCloud",
  "homeTenantId": "...",
  "id": "aaaaaaaaaa",
  "isDefault": true,
  "managedByTenants": [],
  "name": "Azure subscription 1",
  "state": "Disabled",
  "tenantId": "...",
  "user": {
    "name": "***.com",
    "type": "user"
  }
}

$ az ad sp create-for-rbac --name="flixtube6502" --role="Contributor" --scopes"=/subscriptions/aaaaaaaaaa"

{
  "appId": "appId1234",                // service_principal.client_id
  "displayName": "flixtube6502",
  "password": "password1234",        // service_principal.client_secret
  "tenant": "tenant1234"
}
```

내가 사용할 리전에서 지원하는 auzre k8s 버전은 아래에서 확인할 수 있다.

```
$ az aks get-versions --location koreacentral --output table

KubernetesVersion    Upgrades
-------------------  -----------------------
1.24.3               None available
1.24.0               1.24.3
1.23.8               1.24.0, 1.24.3
1.23.5               1.23.8, 1.24.0, 1.24.3
1.22.11              1.23.5, 1.23.8
1.22.6               1.22.11, 1.23.5, 1.23.8
```

```
resource "azurerm_kubernetes_cluster" "cluster" {        // 쿠버네티스 클러스터 리소스 선언
    name                = var.app_name
    location            = var.location
    resource_group_name = azurerm_resource_group.flixtube.name
    dns_prefix          = var.app_name
    kubernetes_version  = "1.24.3"        // 사용할 버전 지정

    linux_profile {        // 쿠버넷 클러스터 인증정보 설정
        admin_username = var.admin_username

        ssh_key {
            key_data = "${trimspace(tls_private_key.key.public_key_openssh)} ${var.admin_username}@azure.com"
        }
    }

    default_node_pool {        // 노드 설정
        name            = "default"
        node_count      = 1
        vm_size         = "Standard_B2ms"
    }

    service_principal {        // azure 인증정보 설정
        client_id     = var.client_id
        client_secret = var.client_secret
    }
}

output "cluster_client_key" {
  value = azurerm_kubernetes_cluster.cluster.kube_config[0].client_key
  sensitve = true
}

output "cluster_client_certificate" {
  value = azurerm_kubernetes_cluster.cluster.kube_config[0].client_certificate
  sensitve = true
}

output "cluster_cluster_ca_certificate" {
  value = azurerm_kubernetes_cluster.cluster.kube_config[0].cluster_ca_certificate
  sensitve = true
}

output "cluster_cluster_username" {
  value = azurerm_kubernetes_cluster.cluster.kube_config[0].username
  sensitve = true
}

output "cluster_cluster_password" {
  value = azurerm_kubernetes_cluster.cluster.kube_config[0].password
  sensitve = true
}

output "cluster_kube_config" {
  value = azurerm_kubernetes_cluster.cluster.kube_config_raw
  sensitve = true
}

output "cluster_host" {
  value = azurerm_kubernetes_cluster.cluster.kube_config[0].host
  sensitve = true
}
```

> 현재 테라폼 최신버전에서 sensitve value를 제한하고 있어 `sensitive=true` 값을 필수로 요구하였다.

생성한 쿠버넷에 접속하기위해 kubectl을 사용한다.

[](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbhScxi%2FbtrNXj5GwrM%2FmF6LNhuITVYYTKOCD1DZy0%2Fimg.jpg)

```
$ brew install kubectl
```

kubectl은 config 파일을 읽어 쿠버넷 클러스터와 접속하는데 기본 경로는 `~/.kube/config` 이다. config 파일엔 각종 인증 키 정보, 토큰 정보가 들어가는데 azure cli를 사용해서 azure k8s의 config 파일을 생성할 수 있다.

```
$ az aks get-credentials --resource-group flixtube6502 --name flixtube6502

$ kubectl version
```

대시보드를 별도로 설치할 수 있다. 대시보드는 기본적으로 클러스터 내부에서만 접속하도록 되어있는데 service, ingress 등을 사용해서 외부에서 접속할 수 있도록 설정해야 외부에서 접속할 수 있다. 여기선 proxy를 사용해 간단하게 접속해본다.

```
// https://github.com/kubernetes/dashboard
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
$ kubectl proxy
// http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/ 에서 접속할수 있다.
```

Full HCL

[main.tf](https://blog.kakaocdn.net/dn/4PhMO/btrNEwiTcD5/tMTRvZOwXQqolhRcw917v1/main.tf?attach=1&knm=tfile.tf)

[variables.tf](https://blog.kakaocdn.net/dn/bWHdqM/btrNExvj2VU/YtfxsYXXpbkvIBzkQqvkSk/variables.tf?attach=1&knm=tfile.tf)

# CD 파이프라인

-   mongodb와 rabbitmq를 쿠버네티스에 배포한다.
-   마이크로서비스를 배포한다.
-   자동화된 CD 파이프라인을 구축한다.

[](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcfpa3B%2FbtrNWrphaK3%2FoWceKsM4OnLpkozCCdEnb0%2Fimg.jpg)

## 지속적인 서비스 제공과 배포

-   CD, Continuous Delivery : 운영 또는 테스트 환경에 업데이트한 코드를 수시로 자동 배포하는 소프트웨어 개발 기술

빠른 개발 주기를 유지하면서 운영 환경에 코드 변경 사항을 신속하고 안전하게 적용해 주는 역할을 한다.

[](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fclkee4%2FbtrNT8qsFGq%2FTMqgygdzoxwmCo78SKrGCK%2Fimg.jpg)

-   코드 변경 사항을 bitbucket repository에 푸시한다.
-   자동 배포 과정이 시작된다.
    -   테라폼 코드를 실행한다.
    -   쿠버넷 클러스터가 호스팅하고 있는 앱을 업데이트한다.

CD는 사실 어떻게 보면 클라우드에서 쉘 스크립트를 자동으로 호출하는 것이다.

## 테라폼으로 컨테이너 배포하기

kubernetes provider를 설정한다.

```
terraform {
  required_providers {
    azurerm = { ... }
    tls = { ... }
    kubernetes = {
      version = "2.13.1"
    }
  }
}

provider "kubernetes" {
    host = azurerm_kubernetes_cluster.cluster.kube_config[0].host

    client_certificate = base64decode(azurerm_kubernetes_cluster.cluster.kube_config[0].client_certificate)
    client_key             = base64decode(azurerm_kubernetes_cluster.cluster.kube_config[0].client_key)
    cluster_ca_certificate = base64decode(azurerm_kubernetes_cluster.cluster.kube_config[0].cluster_ca_certificate)
}
```

> 테라폼 최신 버전에서 provider를 사용하는 구조가 변경되었으므로 최신버전에 맞게 `reuiqred_providers` 블록을 이용해서 버전 등을 선언한다.

> 책의 1.10.0 버전은 darwin\_arm64을 지원하지 않으므로 현재 지원버전인 2.13.1 버전을 사용한다.

kubernetes 클러스터에 mongodb 이미지 배포를 위한 HCL을 작성한다.

```
resource "kubernetes_deployment" "database" {
  metadata {
    name = "database"

    labels = { pod = "database" }
  }

  spec {
    replicas = 1

    selector {
      match_labels = { pod = "database" }
    }

    template {
      metadata {
        labels = { pod = "database" }
      }

      spec {
        container {
          image = "mongo:4.2.8"
          name  = "database"

          port { container_port = 27017 }
        }
      }
    }
  }
}

resource "kubernetes_service" "database" {
    metadata { name = "database" }

    spec {
        selector = {
            pod = kubernetes_deployment.database.metadata[0].labels.pod
        }
        port { port = 27017 }
        type = "LoadBalancer"
    }
}
```

이전에 직접 입력했어야 했던 `client_id`, `client_secret` 값을 인수로 입력한다.

```
# --auto-approve 옵션을 사용해서 yes/no 입력 interaction 없이 바로 적용시킨다. user interaction이 불가능한 CD 플랫폼 등에서 활용될 수 있다. 
$ tf apply -var="client_id=<client_id>" -var="client_secret=<client_secret>" --auto-approve
```

생성된 쿠버넷상의 데이터베이스는 kubectl 명령어를 사용해서 확인할 수 있다.

```
$ kubectl svc

NAME         TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)           AGE
database     LoadBalancer   10.0.178.124   20.249.65.72   27017:30804/TCP   97s
kubernetes   ClusterIP      10.0.0.1       <none>         443/TCP           6m53s
```

똑같이 rabbitMQ도 추가한다.

```
resource "kubernetes_deployment" "rabbit" {
  metadata {
    name = "rabbit"
    labels = { pod = "rabbit" }
  }
  spec {
    replicas = 1
    selector {
      match_labels = { pod = "rabbit" }
    }
    template {
      metadata {
        labels = { pod = "rabbit" }
      }
      spec {
        container {
          image = "rabbitmq:3.8.5-management"
          name  = "rabbit"
          port { container_port = 5672 }
        }
      }
    }
  }
}

resource "kubernetes_service" "rabbit" {
    metadata { name = "rabbit" }
    spec {
        selector = { pod = kubernetes_deployment.rabbit.metadata[0].labels.pod }
        port {
            port        = 5672
            target_port = 5672
        }
    }
}

resource "kubernetes_service" "rabbit_dashboard" {
    metadata { name = "rabbit-dashboard" }
    spec {
        selector = { pod = kubernetes_deployment.rabbit.metadata[0].labels.pod }
        port { port        = 15672 }
        type             = "LoadBalancer"
        # 이 부분에서 dashboard를 LoadBalancer로 외부에 노출하는것은 개발 환경에 한정한다.
        # 내부 인프라를 외부 네트워크에 직접적으로 노출시키는 것은 바람직하지 않다.
    }
}
```

## 테라폼으로 첫 마이크로서비스 배포하기

아래 tf 파일이 기술한 내용이다.

-   이미지를 빌드한다.
-   컨테이너 레지스트리에 로그인을 한다.
-   컨테이너 레지스트리에 이미지를 게시한다.
-   null\_resource : 특정 리소스 타입이 없는 테라폼 리소스를 생성할 때 사용한다.
-   local-exec : 로컬 컴퓨터에서 명령어를 실행한다.

```
# 스크립트에서 사용할 로컬 변수를 선언한다.
locals {
  service_name = "video-streaming"
  login_server = azurerm_container_registry.container_registry.login_server
  username = azurerm_container_registry.container_registry.admin_username
  password = azurerm_container_registry.container_registry.admin_password
  image_tag = "${local.login_server}/${local.service_name}:${var.app_version}"
}

resource "null_resource" "docker_build" {
  triggers = { always_run = timestamp() }
  provisioner "local-exec" {    # command를 실행한다.
    command = "docker build -t ${local.image_tag} --file ../${local.service_name}/Dockerfile-prod ../${local.service_name}"
  }
}

resource "null_resource" "docker_login" {
  depends_on = [ null_resource.docker_build ]    # 이 명령은 null_resource.docker_build 실행 후에 실행된다.
  triggers = { always_run = timestamp() }

  provisioner "local-exec" {
    command = "docker login ${local.login_server} --username ${local.username} --password ${local.password}"
  }
}

resource "null_resource" "docker_push" {
  depends_on = [ null_resource.docker_login ]
  triggers = { always_run = timestamp() }
  provisioner "local-exec" { command = "docker push ${local.image_tag}" }
}

locals {
  dockercreds = {
    auths = {
      "${local.login_server}" = {
        auth = base64encode("${local.username}:${local.password}")
      }
    }
  }
}

resource "kubernetes_secret" "docker_credentials" {
  metadata { name = "docker-credentials" }
  data = { ".dockerconfigjson" = jsonencode(local.dockercreds)}
  type = "kubernetes.io/dockerconfigjson"
}

# video streaming 앱의 배포를 위한 deployment 생성
resource "kubernetes_deployment" "service_deployment" {
  depends_on = [ null_resource.docker_push ]
  metadata {
    name = local.service_name
    labels = { pod = local.service_name }
  }
  spec {
    replicas = 1
    selector { match_labels = { pod = local.service_name } }
    template {
      metadata { labels = { pod = local.service_name } }
      spec {
        container {
          image = local.image_tag
          name  = local.service_name
          env {
            name = "PORT"
            value = "80"
          }
        }
        image_pull_secrets {
          name = kubernetes_secret.docker_credentials.metadata[0].name
        }
      }
    }
  }
}
# video streaming 앱의 네트워크를 위한 service 생성
resource "kubernetes_service" "service" {
  metadata { name = local.service_name }
  spec {
    selector = { pod = kubernetes_deployment.service_deployment.metadata[0].labels.pod }   
    session_affinity = "ClientIP"
    port {
      port    = 80
      target_port = 80
    }
    type       = "LoadBalancer"
  }
}
```

## 비트버킷 파이프라인을 사용한 CD

지금까지 수동으로 실행한 테라폼 스크립트를 자동화하여 CD 파이프라인을 구축한다. 변경사항을 수시로 배포할 수 있으며, 소프트웨어를 배포하는 작업을 자동화함으로써 배포에 소비하는 시간을 필요한 기능을 만드는데 집중할 수 있도록 해준다. 또한 잠재적으로 사람이 하게되는 실수를 크게 줄일 수 있다.

-   비트버킷 파이프라인을 사용하는 이유
    -   하나를 알아두면 다른 것들도 비슷하기 때문에 다루기 쉽다.
    -   아틀라시안 제품군에 속해있어 편하게 사용할 수 있으며 가격이 나쁘지 않다.

배포 쉘 스크립트를 작성한다.

```
set -u # or set -o nounset
: "$VERSION"
: "$ARM_CLIENT_ID"
: "$ARM_CLIENT_SECRET"
: "$ARM_TENANT_ID"
: "$ARM_SUBSCRIPTION_ID"

cd ./scripts
export KUBERNETES_SERVICE_HOST="" # Workaround for https://github.com/terraform-providers/terraform-provider-kubernetes/issues/679
terraform init 
terraform apply -auto-approve \
    -var "app_version=$VERSION" \
    -var "client_id=$ARM_CLIENT_ID" \
    -var "client_secret=$ARM_CLIENT_SECRET"
```

이전에도 언급하였던 데로 테라폼은 로컬폴더의 `terraform.tfstate` 파일을 사용해서 '현재 상태'를 관리한다. tf 파일에 변경이 생겼을 때 CD 플랫폼이 `tf apply` 명령어로 인프라를 업데이트한다면 tfstate 파일이 업데이트 될 것이고 이 파일을 git repository에 업로드해야 현재 상태가 유지되는 것이다. 테라폼은 이 문제를 해결하기 위해 '현재상태'를 저장할 수 있는 별도의 기능을 제공한다.

```
terraform {
  backend "azurerm" {
    resource_group_name  = "flixtube-terraform6502"
    storage_account_name = "flixtubeterraform6502"
    container_name       = "terraform-state"
    key                  = "terraform.tfstate"
  }
}
```

마지막으로 비트버킷의 파이프라인 스크립트를 최상위 폴더에 `bitbucket-pipelines.yaml` 파일로 작성한다. 여기서 사용한 `$BITBUCKET_BUILD_NUMBER` 환경변수는 [비트버킷 파이프라인에서 제공하는 환경변수](https://support.atlassian.com/bitbucket-cloud/docs/variables-and-secrets/)이다.

```
image: hashicorp/terraform:0.12.29
pipelines:
    default:
      - step:
          name: Build and deploy
          services:
            - docker
          script:
            - export VERSION=$BITBUCKET_BUILD_NUMBER
            -  chmod +x ./scripts/deploy.sh
            - ./scripts/deploy.sh
```

애저 인증정보 같은 민감 정보는 비트버킷의 repository에 레포지터리 변수로 기재하면 파이프라인의 변수로 전달된다.

# 마이크로서비스의 자동 테스트

이전까진 데이터베이스에 클라이언트 프로그램을 붙여보거나, 웹브라우저를 열고 주소를 입력해서 응답값이 정상적으로 나오는지 확인하는 방식으로 테스트를 진행했다. 이런 방식은 시간이 많이 걸리므로 자동화가 필요하다.

-   단위/통합/E2E 테스트 예제를 알아본다.
-   테스트를 포함한 파이프라인을 구축한다.

## 새로운 도구

자바스크립트 테스트 도구인 Jest, E2E 테스트를 위한 Cypress를 알아본다. 두가지 모두 자바스크립트로 테스트를 작성하므로 접근성이 좋다.

## 마이크로서비스 테스트

효과적인 테스트는 운영 시스템(시스템 환경, 코드 설정, 사용할 데이터)에 최대한 가까운 조건을 가져야 한다. 이러한 점에서 도커와 도커컴포즈는 운영시스템에 가깝게 환경을 구성해주는 좋은 도구이다. 도커가 정확하게 구성이 잘 상황에서 개발머신에서 잘 동작한다면, 실제 운영 시스템에서도 잘 동작할 것이라 확신할 수 있다.

수동 테스트는 좋은 시작점이며 경험을 쌓을 가치가 있지만, 앱이 고도화되면서 개발을 빠르게 진행하려면 자동화가 필수이다.

## 자동 테스트

마이크로서비스의 테스트 레벨

-   단위 테스트 : 구분된 기능의 개별 코드 함수를 테스트
-   통합 테스트 : 마이크로서비스를 모두 테스트
-   E2E 테스트 : 마이크로서비스 그룹과 프론트엔드를 포함한 앱의 모두를 테스트한다.

[](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FctuGwR%2FbtrNYevIdzQ%2FOz6FQnVJo2QopeDyacy3m1%2Fimg.jpg)

[](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcrXPxe%2FbtrNYqQf0q4%2FrHmvBo2p13Ome8mtNWPPz1%2Fimg.jpg)

자동화된 테스트는 CD와 서로 연결되어 조기 경보 시스템 역할을 한다. 알람이 발생하면 운영 시스템으로 전송하는 것을 중지하고 고객이 문제를 만나는 것을 막을 수 있는 기회를 준다. 초기에 시작하는 것이 이미 구축된 시스템에 적용하는 것 보다 쉽다. 하지만 처음부터 시작한다면 시간과 비용이 많이 들 수 있으니 극초기엔 수동 테스트로 진행하는 것이 더 효율적일 수 있다.

자동화된 테스트는 만병통치약이 아니다. 사람이 직접 하는 수동 테스트의 대안이 될 순 없으므로 수동 테스트는 여전히 필요하다.

## 제스트를 사용한 테스트

제스트는 페이스북에서 만든, 자바스크립트 코드를 테스트할 때 활용할 수 있다. 테스트 코드를 순서대로 로드하고, 테스트할 코드를 실행해서 예상대로 동작했는지 결과를 검증한다. 자바스크립트 테스트 프레임워크로 가장 인기있는 도구이며 빠른 병렬 테스트, 라이브 리로딩이 가능하다.
