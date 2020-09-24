+++
title = "[번역] macOS GUI 앱에 대한 시스템적인 PATH 환경 변수 설정"
date = "2020-09-24"
description = "[번역] Set system-wide PATH environment variable for Mac OS GUI apps"
tags = [
"macOS",
"environment variable"
]
+++

원문 링크: [Set system-wide PATH environment variable for Mac OS GUI apps](https://www.bounga.org/tips/2020/04/07/instructs-mac-os-gui-apps-about-path-environment-variable/)


> 많은 사람들이 Mac OS GUI 앱에서 PATH 환경 변수를 인식하도록 지시하는 방법을 찾는데 어려움을 겪고 있습니다. 때때로 GUI 앱을 설치할 때 시스템에 설치된 바이너리를 사용하려고 하지만 어떤 이유에서인지 찾을 수 없습니다. 이것을 고쳐보겠습니다.  


저는 이 특별한 문제로 인해 한동안 어려움을 겪었는데, 그 첫번째 앱은 [Emacs](https://www.gnu.org/software/emacs/) 였습니다. 어떤 이유에서인지 /usr/bin에 있는 것은 인식했지만 /usr/local/bin은 인식하지 못했습니다. 저는 그게 Emacs와 관련된 것이라고 생각했습니다. 그러다가 [Qute 브라우저를](https://qutebrowser.org/) 설치하고 Qute 내에서 검색중에 또 한번 시스템에서 사용 가능한 일부 바이너리를 인식하지 못하는 일이 발생했습니다.  



약간의 조사를 했지만 유용한 것을 찾지는 못했습니다. 그러다 어느 시점에 터미널에서 GUI 앱 (Emacs, Qute Browser, MacVim 등)을 시작하면 모든 것이 정상이라는 것을 알게 되었습니다. 시스템의 모든 바이너리를 찾을 수 있었습니다.

응용 프로그램 폴더에서 해당 아이콘을 클릭하거나 Spotlight를 사용하여 앱을 시작할 때와 터미널을 사용하여 시작할 때의 동작이 다른 이유는 무엇일까요? 나는 호기심이 생겼습니다. 

이건 정말 진정한 지식을 찾아, 맥 OS 내부의 진정한 마스터에 이르는 아주 먼 길이었습니다.

완전한 이해를 위한 첫 번째 단계는 GUI 앱이 어떻게 내가 쉘에서 설정한 PATH 환경 변수를 인식하고 있는지 내 스스로에게 질문을 던져보는 것이었습니다. 그건 아마도 ~.profile, ~/.bashrc, ~/.bash_profile, ~/.zsh_profile, ~/.zshrc, ~/.config/fish/config.fish 등으로 설정한 내용이죠.

완전한 `PATH`를 알고 있는 셸에서 부터 앱을 시작 해야 한다는 것은 말이 되지 않습니다.  

이 발견 덕분에 확신을 가지고 GUI 앱이 어디에서 경로를 얻는지 이해하기 위해 조사했고 약간 깊이 파고들어간 후에 답을 얻었습니다. 아이콘을 클릭하거나 Spotlight를 통해 시작되는 모든 단일 앱은 launchd 데몬에 의해 설정된 무언가에서 PATH를 가져옵니다.

이제 GUI 앱에서 사용자 정의 한 PATH를 상속받게 하기 위해 다음과 같이 간단하게 /etc/launchd.conf를 변경하면 됩니다.

```shell
setenv PATH /usr/local/bin:/usr/local/sbin:/usr/bin:/bin:/usr/sbin:/sbin
```

이제 /usr/bin 및 /usr/sbin에서만 바이너리를 검색하는 대신 GUI 앱은 다음 모든 경로에서 바이너리를 검색합니다. /usr/ local/bin/usr/local/sbin/usr/bin/bin/usr/sbin/sbin.

컴퓨터를 재부팅하지 않고 변경 사항을 적용하려면 몇 가지 단계를 더 따라야 합니다.  - 저는 컴퓨터를 재부팅하는 것을 싫어합니다. 이 단계를 통해 launchd 및 spotlight가 새로운 설정을 인식하도록 합니다.

```shell
$ egrep "^setenv\ " /etc/launchd.conf | xargs -t -L 1 launchctl
```

이렇게 하면 모든 환경 관련 설정을 launchd로 전달합니다. 그런 다음 Dock 및 Spotlight 앱을 다시 시작해야합니다.

```shell
$ killall Dock
$ killall Spotlight
```

이제 시작하는 모든 GUI 앱은 응용 프로그램 폴더의 아이콘을 클릭하거나 Spotlight를 사용하여 앱을 시작하여도 이 PATH 환경 변수를 상속받습니다. 

Enjoy!
