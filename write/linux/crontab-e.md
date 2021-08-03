# crontab-e 사용하기

리눅스에서 crontab 을 등록하는 방법이 여러가지 이다.

 리눅스 터미널에서 `crontab -e` 를 하면 텍스트 편집기로 crontab에 등록된 작업을 추가/변경/삭제할수 있다. 

만약 쉘이나 프로그램에서 크론탭을 자동으로 수정하고 싶다면?

## cron 추가

아래 명령어는 crontab에 새로운 설정을 추가한다.

```bash
echo '0 0 * * * touch /root/a' >> /var/spool/cron/crontabs/rootservice cron restart
service cron restart
```

  
 아래 방법은 `crontab` 명령어와 stdout을 적절히 조합한 방법이다. cron 재기동이 필요하지 않다.

```bash
{ crontab -l & echo '0 0 * * * 1-6 touch /root/a'; } | crontab -
```

## cron 삭제

 `grep`을 이용하여 특정 설정을 삭제할 수 있다.

```bash
crontab -l | grep -v 'touch /root/a' | crontab -
```

