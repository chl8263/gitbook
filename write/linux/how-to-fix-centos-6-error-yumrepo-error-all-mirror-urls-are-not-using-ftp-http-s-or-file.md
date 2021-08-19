# CENTOS 6 Yum repo 작동이 안될 때

> centos 6 에서 yum update 등 yum 을 사용할 때 아래와 같은 에러가 난다. 이유는 centos6 지원이 끝나서 보안적인 문제로 인한 것.
>
> ```text
> Loaded plugins: fastestmirror, refresh-packagekit, security
> Setting up Update Process
> Determining fastest mirrors
> YumRepo Error: All mirror URLs are not using ftp, http[s] or file.
> Eg. Invalid release/repo/arch combination/
> removing mirrorlist with no valid mirrors: /var/cache/yum/x86_64/6/base/mirrorlist.txt
> Error: Cannot find a valid baseurl for repo: base
> ```

아래 명어를 따라 `CentOS-Base.repo` 를 editer 로 연 뒤

```text
# cd /etc/yum.repos.d/

# cp CentOS-Base.repo CentOS-Base.repo.old

# nano CentOS-Base.repo

```

내용을 아래로 교

```text
[base]
name=CentOS-$releasever - Base
# mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
# baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
baseurl=https://vault.centos.org/6.10/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

# released updates
[updates]
name=CentOS-$releasever - Updates
# mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
# baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
baseurl=https://vault.centos.org/6.10/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

# additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
# mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
# baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
baseurl=https://vault.centos.org/6.10/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
```

그 다음 아래 명령어로 클랜해주고 실행

```text
# yum clean all

# yum update
```

