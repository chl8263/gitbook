# python symbolic link 걸기

### **1. python 실행 파일 위치 및 버전 확인**

```
$ which python
```

필자는 아래와 같이 나온다.

/usr/bin/python

&#x20;

### **2. 기본 python이 어떤 심볼릭 링크에 물려 있는지 확인**

```
$ ls -l /usr/bin/python*
```

![](https://blog.kakaocdn.net/dn/dQ9fJa/btquBK94qf1/LTk3I0MO1k3dheYzVOSY1K/img.png)

위와 같이 python2.7을 가리키고 있다.

&#x20;

**\* 이미 파이썬 버전이 설치되어있다면 3번 생략**&#x20;



### **3. Python 설치**&#x20;

```
$ yum install python2, python3, ..
```



### **4. 심볼릭 링크 걸기**&#x20;

(기존 python 실행 파일 위치인) /usr/bin/python은 지워준다.&#x20;

\* 기존 python 실행 파일 위치 대로 지워준다.&#x20;

```
$ rm /usr/bin/python
```

아래와 같이 심볼릭 링크를 걸어준다.&#x20;

필자는 python3.5 버전을 기본으로 쓰고자 다음과 같이 걸어주었다.

```
$ ln -s /usr/bin/python3.5 /usr/bin/python
```



### **5. 심볼릭 링크 확인**

```
$ ls -l /usr/bin/python*
```

![](https://blog.kakaocdn.net/dn/rmcC2/btquDsUGCaL/ZZHk6qxD1NM0GM9SUOb9Dk/img.png)

기존에 /usr/bin/python 은 python2.7 을 가리키고 있었으나&#x20;

이제 /usr/bin/python3.5 를 가리키고있다. \


\
\
출처: [https://eehoeskrap.tistory.com/316](https://eehoeskrap.tistory.com/316) \[Enough is not enough]
