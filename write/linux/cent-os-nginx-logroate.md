# Cent os / nginx logroate 설정

Nginx log 가 너무 많이 쌓이는 경우를 대비하여 cent os에 logroate 를 사용하여 일정 기간이 지난 로그들을 알아서 지우는 작업을 진행하게 만든다.

{% hint style="info" %}
Linux에서 logrotate 프로그램을 구성하여 로그 파일의 회전, 압축, 삭제 및 메일 링을 자동으로 수행 할 수 있다.

각 로그 파일을 매일, 매주, 매월 또는 너무 커질 때 처리 할 수 ​​있도록 logrotate 프로그램을 구성 할 수 있다.
{% endhint %}

## Logrotate 작동 방식

대부분 logrotate가 기본으로 리눅스 상에 설치가 되어있지만 안되어 있는 경우 따로 설치가 필요하다.

### 동작순서

1. `cron.daily` 에서 `/usr/sbin/logrotate` 호출
2. `/usr/sbin/logrotate` 에서 `/etc/logrotate.conf` 설정파일 참조
3. `/etc/logrotate.conf` 설정 파일에서 `/etc/logrotate.d` 참조 \( `logrotate.conf` 파일 안에 `"include /etc/logrotate.d"`\)

## 작동 순서

### - logrotate 설정

 /etc/logrotate.d/nginx 파일을 만들고 다음 내용 추가한다.

```text
/var/log/nginx/*.log {
        daily
        dateext
        dateyesterday
        missingok
        rotate 365
        notifempty
        create 644 nobody nobody
        sharedscripts
        postrotate
                if [ -f /var/run/nginx.pid ]; then
                        kill -USR1 `cat /var/run/nginx.pid` > /dev/null 2>&1
                fi
        endscript
}
```

* **rotate 365\(숫자\)  : log파일 365개 이상 되면 삭제**
* **maxage 30\(숫자\) : 30일 이산된 로그 파일 삭제**
* **size : 지정한 용량이 되면 로그로테이트를 실행한다. 10k, 10M 이런식으로 지정한다.**
* **create : \[권한 유저 그룹\] 으로 rotation된 로그파일 생성**

{% hint style="danger" %}
실제 nginx 웹 서버의 구동 계정과 맞춰야 하며 잘못 설정했을 경우 nginx 가 로그를 기록할 수 없다.
{% endhint %}

* **notifempty : log 내용이 없으면 rotation 하지 않는다.**
* ifempty : 로그파일이 비어있는 경우에도 로테이트한다.
* monthly : 월 단위로 로테이트 한다.
* daily : 월 단위로 로테이트 한다.
* weekly : 월 단위로 로테이트 한다.
* **compress : rotate 된 로그 gzip 압축**
* nocompress : 압축을 원치 않는다.
* mail [admin@mail](mailto:admin@trdriver.com) : 로테이트 설정에 의해 보관주기가 끝난 파일을 메일로 발송한다.
* mailfirst [admin@mail](mailto:admin@trdriver.com) : 로테이트시 신규파일 이전의 로그를 메일로 발송한다.
* nomail : 메일로 통보받지 않음.
* errors [admin@mail](mailto:admin@trdriver.com) : 로테이트 실행시 에러가 발생하면 이메일로 통보한다.
* prerotate-endscript : 사이의 명령어를 로그파일 처리전에 실행한다.
* postrotate-endscript : 사이의 명령어를 로그파일 처리후에 실행한다.
* extension : 로테이트 후 생성되는 파일의 확정자를 지정한다.
* **copytruncate : 이옵션을 넣지 않으면 현재 사용중인 로그를 다른이름으로 move하고 새로운 파일을 생성한다.**

{% hint style="danger" %}
위의 설정중 `postrotate` 안에는 nginx 를 죽이고 다시 시작해 주어야 새로운 로그 파일에 계속 파일이 쌓이지 않는다. 이 부분은 설정마다 다를 수 있으므로 pid를 참조하는 경로를 찾아준다.
{% endhint %}

## 작동 확인

정상 설정 여부를 테스트하기 위해 다음 명령어 실행한다.

```text
logrotate -d /etc/logrotate.d/nginx
```

위의 명령어를 치면 디버깅 모드로 실제 logrotate를 실행하지 않고 실행계획만 보여준다.

```text
logrotate -f /etc/logrotate.d/nginx
```

따라서 `-f` 옵션을 주어 시간이나 뭐 이런 제약을 무시하고 강제로 실행시키도록 해서 확인해본다.

## Cron 에 등록

여러 블로그에서는 `/etc/cron.daily/logrotate` 에 파일이 있어서 따른 등록 없이 된다고 하는데. 

나는 안되었다. ㄹ

그래서 `/etc/crontab` 에서 cron등록을 따로 해주었다.

```text
SHELL=/bin/bash
PATH=/sbin:/bin:/use/sbin:/use/bin
MAILTO=root

0 0 * * * root /usr/bin/logrotate -f /etc/logrotate.d/nginx
```

그래도 안돈다.... 왜???????????????????

 다 확인해 봤다 크론이 안도는 것도 아니고 크론이 도는데 logrotate 가 제대로 안돈다.. stack over flow 에도 나같은 사람들이 수천가지 이지만 제대로된 해결법을 찾지 못하였다.

`crontab -e` 에 직접 `su` 권한으로 접속 후 등록을 해주었다. `crontab -e`  에 직접 등록을 해주어야 해당 유저의 권한으로 cron이 도는거같다.

아래와 같이 script를 이용해서 cronjob을 등록시킬 수 있다.

```text
(crontab -l 2>/dev/null; echo "*/5 * * * * /path/to/job -with args") | crontab -
```

