# Ubuntu, root 로 GUI 로그인하기

우분투는 GUI에 root 로 한번에 로그인을 할 수 없다. 따라서 아래 설정을 해주자.

### 1. `root` 계정 암호 설정

 우분투는 `root` 로그인 자체를 막아 놨으므로 `root` 계정 암호가 설정돼 있지 않다. `root` 계정으로 로그인하려면 `root` 계정 암호를 설정해야 한다.

```bash
sudo su
passwd
```

#### 2. GDM 설정 <a id="gdm-&#xC124;&#xC815;"></a>

 `/etc/gdm3/custom.conf` 파일을 열여서 `# TimedLoginDelay = 10` 아랫줄에 `AllowRoot=true`라고 라인을 추가.

```bash
# GDM configuration storage
#
# See /usr/share/gdm/gdm.schemas for a list of available options.

[daemon]
# Uncoment the line below to force the login screen to use Xorg
#WaylandEnable=false

# Enabling automatic login
#  AutomaticLoginEnable = true
#  AutomaticLogin = user1

# Enabling timed login
#  TimedLoginEnable = true
#  TimedLogin = user1
#  TimedLoginDelay = 10
AllowRoot=true

[security]

[xdmcp]

[chooser]

[debug]
# Uncomment the line below to turn on debugging
# More verbose logs
# Additionally lets the X server dump core if it crashes
Enable=true
```

### 3. gdm-password 파일 변경

 `/etc/pam.d/gdm-password` 파일에서 아래 코드가 있는 라인 주석.

```bash
auth	required	pam_succeed_if.so user != root quiet_success
```

