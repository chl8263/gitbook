# ICMP timestamp responses disable  하기

리눅스에서 ICMPO time stamp 를 이용해서 서버의 시간을 제공하면, 해커가 서버 시간을 베이스로 활용한ㄴ 공격을 할수 있다. 때문에 ICMP Timestamp 를 막는 것.  &#x20;



Linux 방화벽 설정 : Linux Iptables 에 ICMP timestamp 규칙 삭제 ICMP timestamp Input, Output 규칙 Drop

```bash
[root@ ~]# iptables -A INPUT -p icmp --icmp-type timestamp-request -j DROP
[root@ ~]# iptables -A OUTPUT -p icmp --icmp-type timestamp-reply -j DROP
```

timestamp 규칙이 Drop 으로 변했는지 확인

```bash
[root@ ~]# iptables -L | grep timestamp
```

Iptables 의 Input, Output 규칙 번호 확인

```bash
[root@ ~]# iptables -L INPUT --line-numbers | grep timestamp
[root@ ~]# iptables -L OUTPUT --line-numbers | grep timestamp
```

3번 문항에서 결과로 나온 Iptables Output 규칙 번호를 Iptables 에서 제거

```bash
[root@ ~]# iptables --delete INPUT 6
[root@ ~]# iptables --delete OUTPUT 2
```

reference : [https://www.golinuxcloud.com/disable-icmp-timestamp-responses-in-linux/](https://www.golinuxcloud.com/disable-icmp-timestamp-responses-in-linux/)
