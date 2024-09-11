# GitLab ArgoCD 연동

*   **Argo CD 설치**

    먼저, Argo CD를 설치할 수 있는 Helm 차트 또는 Kubernetes 매니페스트를 사용할 수 있습니다. Helm을 사용하는 방법을 아래에 설명합니다.

    1.  **Helm 저장소 추가**

        ```bash
        helm repo add argo https://argoproj.github.io/argo-helm
        helm repo update
        ```
    2.  **Argo CD 설치**

        ```bash
        helm install argocd argo/argo-cd --namespace argocd --create-namespace
        ```
*   **Argo CD 서비스 접근 설정**

    Argo CD는 기본적으로 내부 서비스로 설정되기 때문에, 외부에서 접근할 수 있도록 Ingress를 설정하거나, NodePort를 설정할 수 있습니다. 예를 들어, NodePort를 사용하는 경우:

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

    Ingress를 사용하는 경우, 적절한 Ingress 리소스를 설정하세요.
*   **Argo CD 로그인 및 초기 설정**

    Argo CD의 기본 비밀번호는 초기 설치 후 자동으로 생성된 비밀번호로 설정됩니다. 이 비밀번호를 얻으려면 다음 명령어를 사용합니다:

    ```bash
    kubectl -n argocd get secret argocd-initial-admin-secret -o yaml
    ```

    비밀번호를 찾은 후, `kubectl port-forward` 명령어를 사용하여 Argo CD UI에 접근할 수 있습니다:

    ```bash
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```

    그런 다음, 브라우저에서 `http://localhost:8080`으로 접속하고, 위에서 얻은 비밀번호로 로그인합니다.
* **Argo CD 사용 시작**
  * Argo CD UI에서 애플리케이션을 정의하고 배포할 수 있습니다.
  * Git 저장소를 연결하고 배포할 애플리케이션을 정의하세요.



Settings- Repository certificates and known hosts를 보면 bitbucket.org와 github와 gitlab 그리고 azure등의 ssh key가 들어가 있다.

&#x20;

그러나 사내에서 구축한 gitlab을 연동하려면 known hosts에 ssh key를 새로 등록해줘야 한다.

&#x20;

아래의 명령어를 사용해서 ssh key를 가져온다.

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

&#x20;

그리고 아래와 같이 Settings- Repository certificates and known hosts에 등록해준다.

<figure><img src="https://blog.kakaocdn.net/dn/cxy0Pw/btsqLR3KbPW/ZFEPQtpb54MhJuyXx5KLFk/img.png" alt=""><figcaption></figcaption></figure>
