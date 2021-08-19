---
description: >-
  yum으로 설치시 불필요하게 설치되는 파일들이 시스템의 필요없는 자원을 사용하게 되고, 관리적으로도 문제가 발생할 수 있다. 때문에
  Linux 에 어떤 모듈을 설치할 때 소스 설치를 하면 더 깔끔하다.
---

# Nginx 소스 설치 하기

### 1. 설치 파일 다운로드

설치파일 url : [http://nginx.org/en/download.html](http://nginx.org/en/download.html)

centos 버전에 호환이 맞는 nginx 버전을 꼭 확인하고 설치받아야 한다. [http://nginx.org/packages/centos](http://nginx.org/packages/centos)에 들어가서 설치하려는 nginx 버전을 확인하는것이 좋다.

### 2. 의존성 다운로드

의존성 파일 버전은 상황에 맞게 다운. 설치하려는 환경 가이드에 없는 의존성 설치가 있을 수 있음

#### - PCRE 인스톨

```text
[root@localhost]# curl -L -O https://ftp.pcre.org/pub/pcre/pcre-8.42.tar.gz
[root@localhost]# tar zxf pcre-8.42.tar.gz
```

#### - zlib 인스톨

```text
[root@localhost]# wget https://zlib.net/zlib-1.2.11.tar.gz
[root@localhost]# tar zxf zlib-1.2.11.tar.gz
```

#### - openssl 인스톨

```text
[root@localhost]# wget https://www.openssl.org/source/openssl-1.1.0h.tar.gz
[root@localhost]# tar zxf openssl-1.1.0h.tar.gz
```

### 3. Nginx 설치

1. `1. 설치 파일 다운로드` 에서 설치한 nginx 디렉토리에 들어간다.

```text
[root@localhost]# cd nginx-1.16.1
[root@localhost nginx-1.16.1]# cd nginx-1.16.1
```

1. configure 설정

본인에 맞는 경로를 수정해서 사용 nginx 에 맞는 configure는상황에 맞게 추가 및 삭제

```text
./configure \
--prefix=/usr/local/ASM4/nginx \
--sbin-path=/usr/local/ASM4/nginx/sbin/nginx \
--conf-path=/usr/local/ASM4/nginx/conf/nginx.conf \
--pid-path=/usr/local/ASM4/nginx/nginx.pid \
--lock-path=/usr/local/ASM4/nginx/logs/nginx.lock \
--error-log-path=/usr/local/ASM4/nginx/logs/error/error.log \
--http-log-path=/usr/local/ASM4/nginx/logs/access/access.log \
--with-pcre=/usr/local/pcre-8.37 \
--with-pcre-jit \
--with-http_addition_module \
--with-http_dav_module \
--with-http_gzip_static_module \
--with-http_realip_module \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-http_sub_module \
--with-http_v2_module \
--with-stream \
--with-openssl=/usr/local/openssl-1.0.2f
```

위의 텍스트를 명령어에 그대로 넣는다.

```text
[root@localhost nginx-1.16.1]# ./configure \
--prefix=/usr/local/ASM4/nginx \
--sbin-path=/usr/local/ASM4/nginx/sbin/nginx \
--conf-path=/usr/local/ASM4/nginx/conf/nginx.conf \
--pid-path=/usr/local/ASM4/nginx/nginx.pid \
--lock-path=/usr/local/ASM4/nginx/logs/nginx.lock \
--error-log-path=/usr/local/ASM4/nginx/logs/error/error.log \
--http-log-path=/usr/local/ASM4/nginx/logs/access/access.log \
--with-pcre=/usr/local/pcre-8.37 \
--with-pcre-jit \
--with-http_addition_module \
--with-http_dav_module \
--with-http_gzip_static_module \
--with-http_realip_module \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-http_sub_module \
--with-http_v2_module \
--with-stream \
--with-openssl=/usr/local/openssl-1.0.2f
```

1. 컴파일

```text
[root@localhost]# make
[root@localhost]# make install
```

