## 노드 OS 업그레이드

커널 업그레이드, libc 업그레이드, 하드웨어 복구 등 노드를 재부팅해야 하는 상황이 있다. 노드를 갑자기 내리면 노드가 장애난 것을 인지하고 다른 노드로 파드을 옮기는 시간(default: 5분)동안 장애가 발생하게 된다. 따라서 아래 절차와 같이 노드를 계획성 있게 내려야 한다.

1. drain node
    - 해당 노드에 있는 파드들을 다른 노드로 이동(배수)시킨다. 만약 다른 노드에 자원이 없다면 pending 상태가 된다.
    - cordon node 명령어가 함께 실행된다. 해당 노드에 POD이 들어올 수도 나갈 수도 없게 (저지선을) 만든다. 해당 노드로 스케쥴링되지 않는다. (SchedulingDisabled)
2. update node
    - 노드를 업그레이드 한다. 1번 실행 시 해당 노드에는 파드가 없기 때문에 안정적으로 업그레이드 가능하다.
3. uncordon node
    - 해당 노드에 다시 파드들이 스케쥴링될 수 있다. 노드를 내릴 때 자원을 할당받지 못한 pending 상태의 파드들은 해당 노드로 배치된다.