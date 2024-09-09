# ImagePullPolicy 를 확인하자

K8S 모든 설정을 수정 했는데도 deploymeny.xml 를 배포해도 spring profiles 가 제대로 설정되지 않았다.

문제는 ImagePullPolicy 설정에 있었다... AKS 기본 ImagePullPolicy 가 **IfNotPresent 였던것 ..**

#### `ImagePullPolicy` 설정

Kubernetes에서 `ImagePullPolicy`를 설정할 때 다음 세 가지 옵션을 사용할 수 있다:

1. **`Always`**: 컨테이너가 시작될 때마다 이미지를 항상 풀어 최신 이미지를 사용한다. 개발 환경에서 유용
2. **`IfNotPresent`**: 이미지가 로컬에 있으면 로컬 이미지를 사용하고, 없으면 이미지를 풀어 사용합니다. 대부분의 프로덕션 환경에서 사용
3. **`Never`**: 이미지를 절대 풀지 않습니다. 로컬 이미지만 사용
