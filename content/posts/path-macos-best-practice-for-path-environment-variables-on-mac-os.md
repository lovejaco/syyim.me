+++
title = "[번역] PATH (MacOS) : Mac OS에서 PATH 환경 변수 모범 사례 "
date = "2020-09-24"
description = "[번역] PATH (MacOS) : Best practice for PATH Environment Variables On Mac OS"
tags = [
"macOS",
"environment variable"
]
+++

원문 링크: [PATH (MacOS) : Best practice for PATH Environment Variables On Mac OS](https://medium.com/@imstudio/path-macos-best-practice-for-path-environment-variables-on-mac-os-35ec4076a486)

> Mac이 훌륭한 OS라는 것을 잘 아시겠지만, MacOS를 사용하다 보면 PATH 환경을 편집하는 방법을 알아야 할 필요가 있습니다. 다행히도 이 작업을 처리하는 것은 쉽습니다.  

## 1. 환경 변수

환경 변수는 윈도, 맥 OS, 리눅스와 같은 운영체제(Operating System)에서 실행되는 모든 프로세스/사용자가 접근할 수 있는 전역 시스템 변수입니다. 환경 변수는 시스템 전체 값을 저장하는 데 유용합니다.

* **PATH:** 가장 자주 사용되는 환경 변수로, 실행 가능한 프로그램을 검색하기 위한 디렉토리 목록을 저장합니다.
* **OS:** 운영 체제.
* **COMPUTENAME, USERNAME:** 컴퓨터와 현재 사용자 이름을 저장합니다.
* **SystemRoot:** 시스템 루트 디렉토리.
* **(Windows) HOMEDRIVE, HOMEPATH:** 현재 사용자의 홈 디렉토리.

## 2. (macOS/Linux) 환경 변수

macOS/Unix의 환경 변수는 대소 문자를 구분합니다. 전역 환경 변수(모든 프로세스에서 사용 가능)의 이름은 대문자로, 단어는 밑줄 (_)로 연결됩니다. (예 : JAVA_HOME) 

지역 변수(현재 프로세스에서만 사용 가능)는 소문자입니다. 권장되는 방법은 `.bash_profile` 파일을 편집하는 것입니다. 이 파일은 시스템에 로그인할 때마다 Bash가 읽고 그 안의 명령을 실행합니다. 가장 좋은 점은 이 파일은 사용자 전용이기 때문에 같은 시스템의 다른 사용자에게 영향을 주지 않는다는 것입니다.

### 2.1 — Bash 셸에서 환경 변수 사용

대부분의 유닉스(Ubuntu/macOS)는 소위 Bash 셸을 사용합니다. 

* 모든 환경 변수를 나열하려면 "env"(또는 "printenv") 명령을 사용하고 "set"을 사용하여 모든 로컬 변수를 포함한 모든 변수를 나열할 수 있습니다.
* 변수를 참조하려면 접두사 '$'와 함께 `$varname`을 사용합니다 (Windows에서는 %varname% 사용).
* 특정 변수의 값을 출력하려면 `echo $varname` 명령을 사용하세요.
* 환경 변수를 설정하려면 변수를 설정하고 전역 환경으로 내보내는 `export varname=value` 명령을 사용합니다 (다른 프로세스에서 사용 가능). 값에 공백이 포함된 경우 값을 큰따옴표로 묶습니다.
* 로컬(지역) 변수를 설정하려면 `varname=value` (또는 `set varname=value`) 명령을 사용합니다. 로컬 변수는 이 프로세스 내에서만 사용할 수 있습니다.
* 로컬(지역) 변수를 해제하려면 `varname=` 명령을 사용합니다. 즉, 빈 문자열로 설정 (또는 `unset varname`)합니다.

### 2.2 — Bash 셸에서 영속적으로 환경 변수를 설정하는 방법

홈 디렉토리의 Bash Shell 시작 스크립트 `~/.bashrc` (또는 "~/.bash_profile" 또는 "~/.profile")에 `export` 명령을 배치하거나 시스템-수준의 작업을 위해 `/etc/profile`을 배치하여 환경 변수를 영속적으로 설정할 수 있습니다.

> 점 (.) 포함된 파일이나 디렉토리는 기본적으로 숨겨져 있습니다. 숨겨진 파일을 표시하려면 "ls -a"또는 "ls -al"명령을 사용하세요. 맥 파인더에서 단축키는 (Shift + Command + .)  

예를 들어 PATH 환경 변수에 디렉토리를 추가하려면 "~/.bashrc" (또는 "~/.bash_profile" 또는 "~/.profile") 끝에 다음 줄을 추가합니다. 여기서 ~(물결 표시는) 현재 사용자의 홈 디렉토리를 의미합니다. 모든 사용자의 경우에는 "/etc/profile”

**1 단계 :** 터미널 창 열기
**2 단계 :** 다음 명령을 입력합니다.

```shell
touch ~/.bash_profile; open ~/.bash_profile
```

이 파일을 통해 실행 중인 환경을 사용자 정의할 수 있습니다.

> Java의 경우) 다음 행을 추가하여 CLASSPATH 환경 변수를 설정할 수 있습니다. 예를 들면  
> export CLASSPATH =. : /usr/local/tomcat/lib/servlet-api.jar  
> Bash 셸은 콜론 (:)을 경로 구분자로 사용합니다. windows는 세미콜론 (;)을 사용합니다.  

**3 단계 :** PATH에 추가 하고 싶은 디렉토리를 경로를 추가하세요. 예를 들면 다음과 같습니다.

```shell
export IM_STUDIO_PATH="$HOME/.imstudio/bin:$PATH"
```

```shell
// Append a directory in front of the existing PATH
export PATH=/usr/local/mysql/bin:$PATH
```

이 예제는 `~/.imstudio` 경로를 PATH에 추가합니다. $PATH 부분은 기존 PATH를 추가하여 새 값으로 유지하므로 매우  중요합니다.

**4 단계 :** .bash_profile 파일을 저장하고 텍스트 편집을 종료합니다 (Command + Q).

> “: wq! nano 편집기를 사용하는 경우”  

**5 단계 :** .bash_profile을 강제 실행합니다. 이렇게하면 재부팅하지 않고 즉시 값을 로드합니다. 터미널 창에서 다음 명령을 실행하십시오.

```shell
source ~/.bashrc
// or
source ~/.bash_profile
source ~/.profile
source /etc/profile
```

이제 Mac OS X 컴퓨터 시스템에서 PATH를 편집하는 방법을 알았습니다. 새 터미널 창을 열고 다음을 실행하여 새 경로를 확인할 수 있습니다.

```shell
echo $PATH
```

이제 PATH에서 원하는 값을 볼 수 있습니다.

**6 단계 :** 마지막으로 PATH에 문제가 있고 잘못된 경로를 "다시 실행"하거나 "제거"하려는 경우 다음과 같이 생각할 수 있습니다.

`launchctl unsetenv IM_STUDIO_PATH`

## 3. Beautiful Tips

### 3.1 - 현재 환경 변수 설정을 확인합니다.

터미널에서 "printenv" 입력

![](https://miro.medium.com/max/1400/1*XmziTfiFd5oYuHTjGzFg1g.png)

### 3.2- 예제 

export JAVA_HOME = $ (/usr/libexec/java_home)
export JRE_HOME = $ (/usr/libexec/java_home)

### 3.3 — Linux / macOS (Bash Shell)에서 JAVA_HOME을 설정하는 방법

먼저 터미널을 시작하고 다음을 실행하여 JAVA_HOME이 이미 설정되어 있는지 확인하십시오.

```shell
echo $ JAVA_HOME
```

`JAVA_HOME` 은 JDK가 설치된 디렉토리로 설정됩니다. JDK가 설치된 디렉토리를 찾아야합니다.

> /Library/Java/JavaVirtualMachines/…  

“~/.bashrc” 끝에 다음 행을 추가하십시오.

```shell
export JAVA_HOME=/path/to/JDK-installed-directory
```

새 설정을 적용하려면 bash 셸을 새로 고쳐야합니다. 다음과 같이 "source"명령을 실행하십시오.

```shell
// Bash 셸 새로 고침
source ~/.bashrc   // or "source ~/.login"
// 새 설정 확인
echo $JAVA_HOME
```

Thank for your watching. Please feel free to feedback for me.

* https://hathaway.cc/2008/06/how-to-edit-your-path-environment-variables-on-mac/
* https://osxdaily.com/2015/07/28/set-enviornment-variables-mac-os-x/
* https://www3.ntu.edu.sg/home/ehchua/programming/howto/Environment_Variables.html

