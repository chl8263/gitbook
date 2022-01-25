# RedHat8 에 Apache openssl 최신버전 적용

RedHat8 부터 system에 있는  openssl 을 교체할 경우 굉장히 위험하고 권장하지 않고있다. 내부 공유라이브러리들이 모두 충돌이나고 깨져버리는 현상이 발생한다. &#x20;

그럼 apache, nginx 등 openssl 을 최신으로 사용하고 싶은 경우는 어떻게 할 수 있을까?

Rocky Linux 에서 system openssl version 은 1.1.1k 지만 openssl 1.1.1m 최신버전을 소스컴파일해 깔고 apache 에 적용하는 부분을 기술해본다.



#### \*\* OS에 기본으로 설치되는 openssl을 변경할 경우 의존성 문제로 명령어 및 프로그램들이 제대로 동작하지 않으므로 변경하면 안 된다.

* Enviroment OS : RockyLinux 8&#x20;
* Openssl : 1.1.1m&#x20;
* apache : 2.4.52

#### 1.openssl 소스설치

```bash
$ cd /usr/local/src
$ wget https://www.openssl.org/source/openssl-1.1.1m.tar.gz
$ tar xvfz openssl-1.1.1m.tar.gz
$ cd openssl-1.1.1
$ ./config --prefix=/usr/local/openssl
$ make && make install
```

#### 2.apache 소스설치

```bash
$ cd /usr/local/src/
$ wget https://archive.apache.org/dist/httpd/httpd-2.4.52.tar.gz
$ wget https://archive.apache.org/dist/apr/apr-1.7.0.tar.bz2
$ wget https://archive.apache.org/dist/apr/apr-util-1.6.1.tar.bz2 
$ wget http://downloads.sourceforge.net/project/pcre/pcre/8.45/pcre-8.45.tar.bz2

$ tar xvf ./httpd-2.4.52.tar.gz 
$ tar xvf ./apr-1.7.0.tar.bz2 && mv ./apr-1.7.0 ./httpd-2.4.52/srclib/apr 
$ tar xvf ./apr-util-1.6.1.tar.bz2 && mv ./apr-util-1.6.1 ./httpd-2.4.52/srclib/apr-util 
$ tar xvf ./pcre-8.45.tar.bz2 

$ yum install -y gcc make gcc-c++ expat-devel perl 
$ yum install -y zlib-devel
$ yum install openssl-devel

$ cd /usr/local/src/
$ tar xvf ./pcre-8.45.tar.bz2
$ cd pcre-8.45
$ ./configure --prefix=/usr/local/pcre-8.45

$ make && make install

$ cd /usr/local/src/httpd-2.4.52
$ ./configure --prefix=/usr/local/apache --enable-mods-shared=all --enable-ext-filter --enable-ssl --with-ssl=/usr/local/openssl --enable-so --enable-cache --enable-proxy --enable-deflate --enable-suexec --enable-file-cache --with-mpm=event --enable-ssl=shared --with-included-apr --with-pcre=/usr/local/pcre-8.45/bin/pcre-config --enable-modules=all --enable-module=shared
$ make && make install 
```

#### 3.apache 서비스 등록

아래 내용을 `/usr/lib/systemd/system` 하위에 `apache.service` 라는 파일을 만들어 내용을 넣는다.

```
[Unit]
Description=The Apache HTTP Server

[Service]
Type=forking
PIDFile=/usr/local/apache/logs/httpd.pid
ExecStart=/usr/local/apache/bin/apachectl start
ExecReload=/usr/local/apache/bin/apachectl graceful
ExecStop=/usr/local/apache/bin/apachectl stop
KillSignal=SIGCONT
PrivateTmp=true
Environment=LD_LIBRARY_PATH=/usr/local/ASM5/openssl/lib

[Install]
WantedBy=multi-user.target
```

```
$ systemctl daemon-reload
$ systemctl enable apache.service
```

#### 4.apache 실행

```
$ systemctl start apache.service
$ systemctl status apache.service // 잘 실행 되었는지 확인
```

#### 5.apache openssl 버전확인

```
$ curl -v 127.0.0.1:80
```

