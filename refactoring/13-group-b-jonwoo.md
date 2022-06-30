# 냄새13. 반복문

-   반복문을 파이프라인으로 바꾸는 리팩토링을 적용하면 필터나 매핑과같은 파이프라인 기능을 사용해 보다 빠르게 어떤 작업을 하는지 파악할 수 있다.

## 반복문을 파이프라인으로 바꾸기

-   콜렉션 파이프라인(java Stream)
-   고전적인 반복문을 파이프라인 오퍼레이션을 상요해 표현하면 코드를 더 명확하게 만들 수 있다.
    -   필터 : true에 해당하는 값만 오퍼레이션
    -   맵 : 전달받은 함수를 사용해 값의 타입을 변환하여 다음 오퍼레이션으로 전달

[https://martinfowler.com/articles/refactoring-pipelines.html](https://martinfowler.com/articles/refactoring-pipelines.html)
