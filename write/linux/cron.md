# Cron : 리눅스 작업 스케줄러

Linux 관리자는 보안이나 시스템 관리 등을 위해 주기적으로 동일한 작업 반복으 수행할 수 있어야 한다.

이를위해 Linux 에서 특정시간에 특정 작업을 수행하는 **cron** 이라는 도구가 있다.

### Cron 의 동작방식

Cron은 시스템이 부팅할 때 시작되고 데몬으로 백그라운드에서 실행된다. 각 크론의 실행주기가 되면 크론이 실행됨.

실행시점에 crontab이란 파일을 읽어서 수행 한다.

###  **Cron 과 crontab의 차이** 

cron과 crontab은 담당 역할이 다르다고 볼 수 있다.

crontab은 스케줄 시간과 실행할 파일의 경로를 관리하고, cron은 crontab을 실행한다.

cron : 실행, 

crontab : 설정

### Crontab 읽는 방법 

예를들어 시스템 전체에 사용되는 crontab인 /etc/crontab 은 아래와 같다.

```text
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```

이 파일은 system crontab 이므로 수정하지 말아야 하고 시스템이 갱신될 때 파일이 교체되므로 수정사항을  잃게 된다. 고로 사용자만의 crontab을 만들어서 사용하도록 하자.

파일의 첫 두 줄은 명령을 실행할 쉘과 프로그램을 검사할 경로를 명시한다. 파일의 나머지는 실제 명령과 예약을 나타낸다.

### Crontab 주기설정

```text
# ┌───────────── min (0 - 59) 
# │ ┌────────────── hour (0 - 23) 
# │ │ ┌─────────────── day of month (1 - 31) 
# │ │ │ ┌──────────────── month (1 - 12) 
# │ │ │ │ ┌───────────────── day of week (0 - 6) (0 to 6 are Sunday to Saturday, or use names; 7 is Sunday, the same as 0) 
# │ │ │ │ │ 
# │ │ │ │ │ 
# * * * * * command to execute
```

 각 별 위치에 따라 주기를 다르게 설정 할 수 있다. 순서대로 **분-시간-일-월-요일** 순이다. 

괄호 안의 숫자 범위 내로 별 대신 입력 할 수 있다.

```text
# 매분 test.sh 실행
* * * * * /home/script/test.sh

# 매주 금요일 오전 5시 45분에 test.sh 를 실행
45 5 * * 5 /home/script/test.sh

# 매일 매시간 0분, 20분, 40분에 test.sh 를 실행
0,20,40 * * * * /home/script/test.sh

# 매일 1시 0분부터 30분까지 매분 tesh.sh 를 실행
0-30 1 * * * /home/script/test.sh

# 매 10분마다 test.sh 를 실행
*/10 * * * * /home/script/test.sh

# 5일에서 6일까지 2시,3시,4시에 매 10분마다 test.sh 를 실행
*/10 2,3,4 5-6 * * /home/script/test.sh
```

### Cron 명령어 사용법

아래의 명령어를 입력하면 현재 사용자별 지정 크론을 입력할 수 있다. 본인이 원하는 크론탭을 바로 위를 참고하여 등록해주면 된다.

```text
$ crontab -e
```

반대로 Crontab에 어떤 내용이 들어있는지는 아래 명령어를 이용하자.

```text
$ crontab -ㅣ
```

Crontab을 지우고 싶다면 아래와 같은 명령어 사용.

```text
$ crontab -r
```

## Cron.d 를 이용한 cron 등록

### /etc/cron.d :  `소프트웨어 패키지를 설치할 때 필요한 주기적 작업을 등록하는 공간으로 사용.`

이것을 이용해서 cron 을 간편하게 등록할 수 도 있다.

아래사진과 같이 /etc/cron.d 디렉토리에 **apache\_add.cron** 이라는 사용자가 원하는 cron 파일을 등록할 수 있다.

![](../../.gitbook/assets/image%20%282%29.png)

\[**apache\_add.cron**\] 예

```text
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

42 9 * * * root sh /usr/local/test.sh
```

이때 /etc/cron.d 에 등록되는 파일의 권한이 낮으면 낮을 수 록 좋다. 

#### 권한이 700 번대로 넘어가면 Cron 이 자체적으로 위험할 수 있는 파일 이라고 인식하여 돌리지 않음

#### - 취약점 점검기준 -

* 양호기준: cron 접근제어 파일 소유자가 root이고, 권한이 600 이하인 경우 양호
* 취약기준: cron 접근제어 파일 소유자가 root가 아니거나, 권한이 600 이하가 아닌 경우 취약

