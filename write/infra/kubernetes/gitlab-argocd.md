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

