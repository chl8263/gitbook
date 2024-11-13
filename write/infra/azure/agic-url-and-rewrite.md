# AGIC의 Url & Rewrite

AKS 와 AGIC로   구성되어있는환경에서

만약 `dev.ewan.com` 으로 통일된 도메인에 뒤에 path 로 AKS 서비스를 구분하여 전송하고 싶다

ex)

* dev.ewan.com/service1
* dev.ewan.com/service2
* dev.ewan.com/service3

이렇게 관리하기 위해서는 rewrite 규칙을 설정해야함

ex)&#x20;

`http://dev.ewan.com/service1/api/v1/domain`&#x20;

\->  `http://dev.ewan.com/api/v1/domain`

url 경로 중간에 `service1` 를 제거하고 나머지 부분을 application 에 전달해야한다.

* application gateway 에 수동으로 rewrite 룰을 작성하고 [https://learn.microsoft.com/ko-kr/azure/application-gateway/rewrite-url-portal](https://learn.microsoft.com/ko-kr/azure/application-gateway/rewrite-url-portal)
* azure portal 에서 설정한 rewrite 네임을 ingress.yaml 에 설정

```Yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ...
  namespace: ...
  annotations:
    appgw.ingress.kubernetes.io/health-probe-path: "/health"
    appgw.ingress.kubernetes.io/rewrite-rule-set: service1-rewrite # azure portal 에서 설정한 rewrite 값을 설정
```

\
yaml 로 azure portal 에  수동으로작업하지않고 구성하고 싶었으나 수많은 시도 끝에도 결국 방법을 찾지 못했음.[https://stackoverflow.com/questions/78705021/azure-application-gateway-equivalent-for-url-rewriting-in-nginx-ingress](https://stackoverflow.com/questions/78705021/azure-application-gateway-equivalent-for-url-rewriting-in-nginx-ingress)[https://azure.github.io/application-gateway-kubernetes-ingress/features/rewrite-rule-set-custom-resource/](https://azure.github.io/application-gateway-kubernetes-ingress/features/rewrite-rule-set-custom-resource/)
