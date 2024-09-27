# GitLab ArgoCD 연동

## **Argo CD 설치**

먼저, Argo CD를 설치할 수 있는 Helm 차트 또는 Kubernetes 매니페스트를 사용할 수 있다.&#x20;

Helm을 사용하는 방법을 아래에 설명한다.

1.  **Helm 저장소 추가**

    ```bash
    helm repo add argo https://argoproj.github.io/argo-helm
    helm repo update
    ```
2.  **Argo CD 설치**

    ```bash
    helm install argocd argo/argo-cd --namespace argocd --create-namespace
    ```

## **Argo CD 서비스 접근 설정**

Argo CD는 기본적으로 내부 서비스로 설정되기 때문에, 외부에서 접근할 수 있도록 Ingress를 설정하거나, NodePort를 설정할 수 있다. 예를 들어, NodePort를 사용하는 경우:

```yaml
apiVersion: services/v1
kind: Service
metadata:
  name: argocd-server
  namespace: argocd
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
  selector:
    app.kubernetes.io/name: argocd-server
```

Ingress를 사용하는 경우, 적절한 Ingress 리소스를 설정.

## **Argo CD 로그인 및 초기 설정**

Argo CD의 기본 비밀번호는 초기 설치 후 자동으로 생성된 비밀번호로 설정된다. 이 비밀번호를 얻으려면 다음 명령어를 사용:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o yaml
```

비밀번호를 찾은 후, `kubectl port-forward` 명령어를 사용하여 Argo CD UI에 접근할 수 있다:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

그런 다음, 브라우저에서 `http://localhost:8080`으로 접속하고, ID는 `admin` , 비번은 위에서 얻은 비밀번호로 로그인한다.

## **Argo CD 사용 시작**

* Argo CD UI에서 애플리케이션을 정의하고 배포할 수 있다.
* Git 저장소를 연결하고 배포할 애플리케이션을 생성한다.
* Settings- Repository certificates and known hosts를 보면 bitbucket.org와 github와 gitlab 그리고 azure등의 ssh key가 들어가 있다.
* 사내에서 구축한 gitlab을 연동하려면 known hosts에 ssh key를 새로 등록해줘야 한다.
* 아래의 명령어를 사용해서 ssh key를 가져온다.

```
$ ssh-keyscan gitlab.somaz.link
# gitlab.somaz.link:22 SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3
gitlab.somaz.link ssh-rsa AAAAB3NzaC1y...
# gitlab.somaz.link:22 SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3
gitlab.somaz.link ecdsa-sha2-nistp256 AAAAE2V...
# gitlab.nerdystar.io:22 SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3
gitlab.somaz.link ssh-ed25519 AAAAC3...
# gitlab.somaz.link:22 SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3
# gitlab.somaz.link:22 SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3
```

그리고 아래와 같이 Settings- Repository certificates and known hosts에 등록해준다.

<figure><img src="https://blog.kakaocdn.net/dn/cxy0Pw/btsqLR3KbPW/ZFEPQtpb54MhJuyXx5KLFk/img.png" alt=""><figcaption></figcaption></figure>

## 키 설정

* 사용자 로컬에서 `ssh-keygen -t ed25519 -f argocd` 명령어로 공개키 및 개인키를 미리 생성해놓는다.
* 연결할 gitlab repository 에서 settngs-Repository 에 들어가 Deploy key 를 새로 설정하는데, 위에 생성한 공개키를 넣어준다.

<figure><img src="../../../.gitbook/assets/Screenshot 2024-09-12 at 9.50.33 AM.png" alt=""><figcaption></figcaption></figure>

* 생성된 개인키는 아래 화면처럼 argocd repo 설정 화면에 `ssh private key data` 에 넣고 connect 버튼을 누른다.

<figure><img src="../../../.gitbook/assets/Screenshot 2024-09-12 at 9.54.04 AM.png" alt=""><figcaption></figcaption></figure>

## 4. Application 설정

* Application - new app 을 눌러 화면 진입
*

    <figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>


*

    <figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

#### 1. General 설정:

* Application Name: 배포할 애플리케이션의 이름을 지정
* Project: 애플리케이션이 속할 프로젝트를 선택 (기본값은 `default`).
* Sync Policy: 자동 동기화를 설정할지, 수동으로 할지를 결정
* SYNC OPTIONS
  * SKIP SCHEMA VALIDATION : 매니패스트에 대한 yaml 스키마 유효성 검사를 건너뛰고 배포 (kubectl apply --validate=false)
  * PRUNE LAST : 동기화 작업이 끝난 이후에 Prune(git에 없는 리소스를 제거하는 작업)를 동작시킴
  * RESPECT IGNORE DIFFERENCES : 동기화 상태에서 특정 상태의 필드를 무시하도록 함
  * AUTO-CREATE NAMESPACE : 클러스터에 네임스페이스가 없을 시 argocd에 입력한 이름으로 자동 생성
  * APPLY OUT OF SYNC ONLY : 현재 동기화 상태가 아닌 리소스만 배포
  * SERVER-SIDE APPLY : 쿠버네티스 서버에서 제공하는 Server-side Apply API 기능 활성화 (레퍼런스 참조)
* PRUNE PROPAGATION POLICY ​(레퍼런스 참조)
  * foreground : 부모(소유자, ex. deployment) 자원을 먼저 삭제함
  * background : 자식(종속자, ex. pod) 자원을 먼저 삭제함
  * orphan : 고아(소유자는 삭제됐지만, 종속자가 삭제되지 않은 경우) 자원을 삭제함



#### 2. Sorce 설정:

* Repository URL: GitLab 리포지토리의 URL을 입력

예: https://gitlab.com/your-username/your-repo.git

* Revision: `HEAD` 또는 특정 Git 브랜치(예: `main` 혹은 `master`)를 입력
* Path: GitLab 리포지토리 내 Helm 차트가 위치한 경로를 입력. 예를 들어, 차트가 `helm/` 디렉토리 안에 있다면 `helm`을 입력



#### 3.Destination 설정:

* Cluster URL: 애플리케이션을 배포할 Kubernetes 클러스터의 URL이다. 기본적으로 [https://kubernetes.default.svc/](https://kubernetes.default.svc/)가 사용
* Namespace: 애플리케이션이 배포될 네임스페이스를 입력. (예: `default` 혹은 원하는 네임스페이스



#### 4. Application 생성 후 배포:

* 모든 설정을 완료한 후 "Create" 버튼을 눌러 애플리케이션을 생성.
* 생성된 후, ArgoCD UI의 애플리케이션 목록에서 새로 생성된 애플리케이션을 선택하면, `Sync` 버튼을 눌러 수동으로 동기화를 하거나 자동으로 배포되도록 설정한 경우 ArgoCD가 이를 처리한다\


#### 5. ArgoCD에서 배포 상태 확인:

* 애플리케이션이 생성되면 ArgoCD UI에서 해당 애플리케이션의 상태를 실시간으로 모니터링할 수 있다.
* Sync 상태, Health 상태 등이 표시되며, 문제가 발생하면 UI에서 직접 문제를 해결할 수 있다.



#### 6. Webhook 설정 (Optional)

* GitLab에서 코드 변경 사항이 발생했을 때 자동으로 ArgoCD가 이를 감지하게 하려면 GitLab Webhook을 설정할 수 있다. 이를 통해 GitLab에서의 푸시나 머지 요청 시 자동으로 배포를 트리거할 수 있다.
* GitLab Webhook 설정:
* GitLab 리포지토리의 Settings > Webhooks에서 새로운 Webhook을 추가
* Webhook URL은 ArgoCD의 `https://<argocd-server>/api/webhook` 형식으로 설정
* 필요한 이벤트를 선택하고 Webhook을 저장
