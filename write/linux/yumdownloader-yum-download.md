# yumdownloader 명령어로 yum 패키지를 설치하지 않고 download 만 하기

패키지를 설치해야 하는 서버가 보안등의 이유로 인터넷이 안 될 경우 _yum install_ 명령어로 직접 설치할 수 없습니다.

이럴 경우 인터넷이 되는 서버에서 rpm 패키지를 다운로드받은 후에 설치 서버로 이동해서 설치해야 하며 이때 패키지 바이너리나 소스를 다운로드 받을수 있는 _yumdownloader_ 명령어를 사용하면 됩니다.

### 설 <a href="#yumdownloader-yum-download" id="yumdownloader-yum-download"></a>

_yumdownloader_ 명령어가 있는 _yum-utils_ 패키지를 설치합니다.

```bash
$ sudo yum install yum-utils
```

### 사 <a href="#yumdownloader-yum-download" id="yumdownloader-yum-download"></a>

_yumdownloader_ 명령어에 --_downloadonly_ 옵션을 주고 다운받을 패키지를 지정하면 현재 폴더에 다운로드됩니다.

```bash
$ yumdownloader --downloadonly gcc
```

\


하지만 패키지는 보통 다른 패키지에 의존하는 경우가 많습니다. 이런 의존성 정보는 _yum deplist_ 명령어로 확인할 수 있으며 아래는 gcc 가 의존하는 패키지 목록입니다.

```bash
$ yum deplist  gcc

package: gcc.x86_64 4.4.7-23.el6
  dependency: libgomp = 4.4.7-23.el6
   provider: libgomp.x86_64 4.4.7-23.el6
   provider: libgomp.i686 4.4.7-23.el6
  dependency: libgomp.so.1()(64bit)
   provider: libgomp.x86_64 4.4.7-23.el6
 ...
  dependency: libgcc_s.so.1()(64bit)
   provider: libgcc.x86_64 4.4.7-23.el6
```

_yumdownloader_ 명령어에 --_resolve_ 옵션을 추가하면 의존성 있는 패키지도 같이 다운로드합니다.

```
yumdownloader --downloadonly --resolve gcc
```

ㄴㅇREF

[https://www.lesstif.com/system-admin/yumdownloader-yum-download-100205937.html](https://www.lesstif.com/system-admin/yumdownloader-yum-download-100205937.html)

\
