# python window service 에서 다른 process 띄우기

Windows 운영 체제에서는 다양한 사용자들이 동시에 컴퓨터를 사용할 수 있다.

각 사용자는 각자의 작업을 수행하는 독립된 세션을 가지고 있다. 이러한 세션은 로그인한 사용자마다 생성되며, 각각의 세션은 사용자의 작업 환경을 유지하고 해당 사용자에게 할당된 데스크톱을 관리한다.

### Windows 서비스는 백그라운드에서 실행되며, 시스템의 로그인 화면에서 로그인한 사용자의 세션과는 별개로 실행된다.

### 따라서 윈도우 서비스가 사용자의 GUI 환경에 직접적으로 액세스할 수 없다.&#x20;

### 서비스에서 GUI 프로그램을 실행하려면 사용자의 세션에 액세스해야 한다.



## ! 따라서 win32process.CreateProcessAsUser 을사용하여 프로세스를 띄워야 함

Windows 서비스에서 GUI 프로그램을 띄우려면 `win32process.CreateProcessAsUser`를 사용해야 하는 자세한 이유:

1. **권한 문제 해결**&#x20;
   * Windows 서비스는 일반적으로 백그라운드에서 실행되는데, 이는 GUI 프로그램을 실행하기에 적합하지 않다
   * 사용자 세션과 관련된 GUI 작업을 수행하기 위해서는 사용자의 컨텍스트에서 프로세스를 실행해야 함
   * `CreateProcessAsUser`를 사용하면 특정 사용자의 컨텍스트에서 프로세스를 실행할 수 있으므로, 사용자의 GUI 세션에 액세스할 수 있다
2. **세션 및 데스크톱 지정**
   * `CreateProcessAsUser`를 사용하면 실행할 프로세스의 세션 및 데스크톱을 명시적으로 지정할 수 있다
   * GUI 프로그램을 실행할 때 사용자의 세션과 데스크톱을 지정해야 하므로, 이러한 기능이 필요
3. **서비스 백그라운드에서 실행**
   * Windows 서비스는 백그라운드에서 실행되며, 일반적으로 사용자의 GUI 환경에 액세스할 수 없다.&#x20;
   * 따라서 사용자의 컨텍스트에서 프로세스를 실행해야만 GUI를 사용할 수 있다.

요약하면, `win32process.CreateProcessAsUser`를 사용하면 Windows 서비스에서 GUI 프로그램을 사용자의 컨텍스트에서 실행할 수 있으며, 이는 사용자 세션과 관련된 작업을 수행하기 위해 필요함



### 사용 예

```python
console_user_token = win32ts.WTSQueryUserToken(process.SessionId)
my_app_path = "myapp/path/example.exe"
startup = win32process.STARTUPINFO()
priority = win32con.NORMAL_PRIORITY_CLASS

environment = win32profile.CreateEnvironmentBlock(console_user_token, False)

handle, thread_id, pid, tid = win32process.CreateProcessAsUser(console_user_token,
                                                                my_app_path,
                                                                None, None,
                                                                None, True, priority,
                                                                environment,
                                                                None, # <- 현재 작업 디렉터리를 명시적으로 지정해주는 것이 의도치 않은 working directory 참조를 피할 수 있게 해주기 때문에 명시적으로 넣는 것을 추천
                                                                startup)
```

