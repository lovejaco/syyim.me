+++
title = "vscode를 커맨드라인으로 실행할 때 Dock 아이콘이 중복으로 생성되는 문제 해결"
date = "2019-05-13"
tags = [
"tips",
"vscode"
]
+++

먼저, 이 증상은 macOS Mojave(그 이상)에서 나타나는 현상입니다.

### 커맨드라인 명령으로 code 실행

쉘의 환경 설정에 path를 지정하고, 확인합니다.

```bash
> nano ~/.zshrc
export PATH=/Applications/Visual\\ Studio\\ Code.app/Contents/Resources/app/bin:$PATH

> which code
/Applications/Visual\ Studio\ Code.app/Contents/Resources/app/bin/code

> code {경로}
```

그런 다음, 커맨드 라인 명령으로 vscode를 실행할 수 있습니다.

### dock 아이콘 중복 생성

code를 실행할 때마다 dock 아이콘이 중복해서 생성됩니다.
![](/images/screen-shot-2020-05-12.png)

### macOS Mojave: code command line generate duplicate icons in dock #60579

vscode의 github 리파지토리에서 [관련 이슈](https://github.com/microsoft/vscode/issues/60579)를 발견 하였습니다. 그 중 제가 확인한 방법은 다음과 같습니다.

#### 1. 시스템 설정 변경

최근 사용한 응용프로그램 보기 **비 활성**

> System Preference > Dock > Show recent applications inDock

[링크(macOS Mojave: Turn Off Recent Applications to Remove Extra Dock Icons)](https://www.techjunkie.com/recent-applications-extra-icons-mojave-dock/)를 참조하여 최근 사용한 애플리케이션 항목을 비 활성할 수 있습니다.

![https://user-images.githubusercontent.com/900690/69153851-3d42e380-0adf-11ea-88a2-b063875e5124.gif](https://user-images.githubusercontent.com/900690/69153851-3d42e380-0adf-11ea-88a2-b063875e5124.gif)

이 방법을 적용하면, 아이콘이 나타났다가 사라지는 애니메이션 때문에 어딘가 좀 어색합니다. 그리고 개인적으로 시스템 설정을 변경하는 것을 원하지 않았습니다.

#### 2. Define Shell function

.bashrc 또는 .zshrc 같은 shell의 환결 설정에 함수를 지정하는 방법이 있습니다.
저는 다음의 조치를 따랐습니다.

```bash
> nano ~/.zshrc
code() {
    if [ -t 1 ] && [ -t 0 ]; then
        open -a Visual\\ Studio\\ Code.app "$@"
    else
        open -a Visual\\ Studio\\ Code.app -f
    fi
}
```

vscode를 모두 종료하고 터미널을 새로 열어 테스트 하니 잘 동작합니다.
