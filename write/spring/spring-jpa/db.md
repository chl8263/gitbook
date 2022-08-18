# DB 메모리 누수

mysql은 wait\_timout(기본값 8시간)동안 접속하지 않으면 커넥션 연결을 종료하여 이후에 종료된 커넥션에 접근하여 RecoverableDataAccessException에러가 생긴다.



시스템에선 connection pool 이 active 하다고 판단하는데 dbms 에서는 connection 만료로 다 날아가버렸었습니다.&#x20;

Client가 한시도 쉬지 않는 장비(장비당 1초에 5번)들과의 통신이었는데요. 네트워크 망 트러블로 단절이 났었어요. 당연히 해당 관련 부분 장애 기록(위 로그랑 거의 같네요)을 한가득 받았습니다.

세상에 절대란 없다 생각하시고 일정부분 자원을 희생하는게 안정성에 더 무게를 주실수 있는 방안 같습니다. 덧. 저는 관련 세팅해놔도 잘 안되서 스케쥴링으로 블리딩 패킷까지 날리던 상황까지 갔었습니다.

몇분 혹은 몇시간에 한번 SELECT 1 날린다고 아까워하시면 나중에 더 크게 돌려받습니다.



[https://jaehun2841.github.io/2020/01/08/2020-01-08-hikari-pool-validate-connection/#maxLifetime-wait-timeout-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%84%A4%EC%A0%95%ED%95%B4%EC%95%BC-%ED%95%98%EB%82%98%EC%9A%94](https://jaehun2841.github.io/2020/01/08/2020-01-08-hikari-pool-validate-connection/#maxLifetime-wait-timeout-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%84%A4%EC%A0%95%ED%95%B4%EC%95%BC-%ED%95%98%EB%82%98%EC%9A%94)

{% embed url="https://okky.kr/article/693424" %}

[https://do-study.tistory.com/97](https://do-study.tistory.com/97)
