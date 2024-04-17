# 파이썬에서 외부 파일을 실행하기 위해서 사용하는 명령어

* ### os.system("실행할 파일")
* ### os.popen("실행할 파일")
* ### subprocess.call("실행할 파일")

이 3가지 중 os 모듈을 이용하는 것은 실행한 파일이 종료되기 전까지는 계속 메모리에 상주.\
os.system과 os.popen은 cmd에서 명령어를 입력하는 것과 같은 동작.\
즉, "실행할 파일"을 구동시키게 되면 실행된 것들의 프로세싱이 끝나기 전 까지는 프로세스로서 cmd.exe가 메모리에 상주.

subprocess만이 cmd를 통하지 않고 바로 실행을 시켜주지만 여전히 "실행할 파일"은 본인을 실행시킨 프로세스에 자식 프로세스로 귀속.\
만약 내가 어떤 외부 파일을 실행하고자할때 잠깐 구동되고 종료되는 거라면야 뭐 cmd도 같이 종료가 되니 크게 문제될 것은 없겠지만 위와 같이 아예 창을 띄우는 프로그램을 구동시켜야 하는 경우에는 얘기가 달라집니다. 창이 띄워져 있는 동안은 cmd도 같이 부모 프로세스로 메모리를 차지한다.\
(subprocess 또한 cmd는 안 띄우지만 python.exe가 부모 프로세스로서 메모리를 차지함.)

### 이럴 때는 Windows API인 ShellExecuteA 나 ShellExecuteEx 메소드들을 이용하면됩니다.Windows API이기 때문에 파이썬에서는 ctypes 모듈을 이용하여 사용.

```
import ctypes
calc = ("c:\windows\system32\\calc.exe")
notepad = ("c:\windows\system32\\notepad.exe")
paint = ("c:\windows\system32\\mspaint.exe")
 
ctypes.windll.shell32.ShellExecuteA(0, 'open', calc, None, None, 1)
ctypes.windll.shell32.ShellExecuteA(0, 'open', notepad, None, None, 1)
ctypes.windll.shell32.ShellExecuteA(0, 'open', paint,  None, None, 1)
```

Windows API인 ShellExecuteA 나 ShellExecuteEx 메소드들을 이용하면 3개의 프로그램 모두 독립적으로 실행.\
실제 Process Monitoring 툴을 이용하여 확인을 해봐도 cmd.exe의 반응은 전혀 일어나지 않고, 부모 프로세스 없이\
독립적으로 실행이 되는것을 확인할수 있다.\
외부의 어떤 파일을 구동하기 위한 작업에는 Windows API인 ShellExecuteA 나 ShellExecuteEx 메소드들을 이용 가장 적합.

ShellExecute의 매개변수가 유니코드를 사용하는 타입이면 ShellExecuteW를 사용하고, 유니코드가 아니라면 ShellExecuteA를 사용하는 것이다.

* ShellExecuteW \~ 유니코드
* ShellExecuteA \~ 유니코드 아닌 것
