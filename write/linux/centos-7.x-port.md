# CentOS 7.x Port 방화벽 관리

아래와 같이 centos 에서 iptables 명령어를 사용해

```bash
[root@vultr ~]# iptables -I INPUT -p tcp --dport 5432 -j ACCEPT
```

아래와 같이 iptables 서비스를 재시작하면 해당 서비스를 사용할 수 없다고 나온다.

```text
[root@vultr ~]# service iptables restart
Redirecting to /bin/systemctl restart iptables.service
Failed to restart iptables.service: Unit not found.
```

centos 7 에서는 iptables 대신에 `firewall` 명령어를 사용한다.

1. 방화벽 상태 확인

   ```text
   [root@vultr ~]# firewall-cmd --state
   ```

2. port 추가

   ```text
   [root@vultr ~]# firewall-cmd --permanent --add-port=21/tcp
   ```

3. port 삭제

   ```text
   [root@vultr ~]# firewall-cmd --permanent --zone=webserver --remove-port=9090-9100/tcp 
   ```

4. 설정내용 reload

   ```text
   [root@vultr ~]# firewall-cmd --reload
   ```

