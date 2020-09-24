+++
title = "Sourcetree(osx)에서 Husky(git-commit-hook)가 동작하지 않는 이유: Can’t find “…” in PATH"
date = "2020-09-24"
description = "Sourcetree(osx)에서 Husky(git-commit-hook)가 동작하지 않는 이유: Can’t find “…” in PATH"
tags = [
"macOS",
"Husky",
"Sourcetree"
]
+++

사실 이 현상은 Husky나 Sourcetree의 문제가 아니라 Mac GUI 앱에서 PATH 환경변수를 올바르게 인식하지 못하는 문제로, 다른 앱에서도 유사한 현상이 발생할 수 있습니다. 이 게시물에서는 이와 같은 현상이 왜 발생하며 어떻게 조치해야 하는지에 대해서 설명 합니다.  

## Husky 

최근 합류한 프로젝트에서는 [Husky](https://www.npmjs.com/package/husky)를 도입하여 git-commit-hook에서 노드 버전을 고정하거나, Lint와 같은 여러 사전 작업을 수행하고 검사합니다. 

저는 주로 터미널을 이용하여 Git 커맨드를 수행하지만 가끔씩 Sourcetre를 사용하기도 하는데 터미널에서 커밋을 작성하면 잘 동작하지만, Sourcetree에서는 hook이 실패하는 현상이 발생하였습니다. 

**Sourcetree > View > Show Command History**

```shell
.huskyrc: line 9: nvm: command not found
env: node: No such file or directory
Can't find Husky, skipping pre-commit hook
You can reinstall it using 'npm install husky --save-dev' or delete this hook
Completed successfully
```

노드 버전을 고정하기 위한 사용하는 nvm 커맨드를 찾을 수 없음

### Husky 이슈 리포트

[Husky Repo](https://github.com/typicode/husky/issues/390)에 올라온 유사한 이슈에서 프로젝트 오너는 다음과 같이 설명 합니다.

> 그것은 실제로는 오류가 아니라 정보 메시지 입니다. 그러나 출력이 빨간색이므로 오류처럼 보일 수 있습니다.  
>
> 노드 버전 관리자(nvm)는 터미널이 시작될 때 PATH를 수정하여 동작합니다. 일반적으로 GUI 클라이언트는 nvm이 초기화를 위해 .bashrc 또는.zshrc 설정을 적용하는 source 명령을 실행하지 않기 때문에 nvm과 잘 동작하지 않습니다.   
>
> 따라서 이 메시지는 Husky가 노드 바이너리를 찾으려는 모드로 진입할 것이라는 것을 알려주기 위한 것입니다 (  [run-node](https://github.com/sindresorhus/run-node)을 사용합니다.)  

[Husky 공식 문서](https://github.com/typicode/husky#node-version-managers)에는 다음과 같이 설명 되어 있습니다. 

#### Node version managers

> Windows를 사용하는 경우 husky는 시스템에 전역적으로 설치된 버전을 사용합니다.  
>
> macOS 및 Linux 사용자의 경우 :  
> 터미널에서 git 명령을 실행하는 경우 husky는 Shell의 PATH에 정의된 버전을 사용합니다. 즉, nvm 사용자 인 경우 husky는 nvm으로 설정한 노드 버전을 사용합니다.  
>
> GUI 클라이언트와 함께 nvm을 사용하는 경우에는 PATH가 달라서 nvm을 로드하지 않을 수 있습니다. 이 경우 일반적으로 nvm에 의해 설치된 가장 높은 노드 버전이 선택됩니다. 또한 ~/.node_path를 확인하여 GUI에서 사용하는 버전을 확인하거나 다른 것을 사용하려는 경우 편집할 수도 있습니다.  

그러니까 macOS에서 GUI 클라이언트를 사용하면 일반적으로 .bashrc 또는.zshrc로 설정한 사용자 지정 환경 변수들을 인식하지 않기 때문에 Shell의 환경과 다를 수 있습니다

```shell
Can't find yarn in PATH: /Applications/Xcode.app/Contents/Developer/usr/libexec/git-core:/Applications/Sourcetree.app/Contents/Resources/bin:/usr/bin:/Applications/Sourcetree.app/Contents/Resources/git_local/gitflow:/Applications/Sourcetree.app/Contents/Resources/git_local/git-lfs:/usr/bin:/bin:/usr/sbin:/sbin
Skipping pre-commit hook
```

 `.huskyrc`  파일을 삭제하고 테스트를 진행하면 역시나 PATH를 찾지 못하는 것을 확인할 수 있음.


Husky를 사용 중이며 Sourcetree에서 nvm을 로드할 수 있도록 조치해야 한다면 가장 보편적으로 수행하는 방법은 `.huskyrc` 에서 PATH 환경 변수를 설정해야 합니다.-  [참고](https://github.com/typicode/husky/issues/390#issuecomment-545855628)

> 그러나 사용자에 따라 PATH가 동일하지 않을 수 있습니다. 예를 들면, 설치된 방법에 따라 PATH가 다를 수 있기 때문에 자신의 환경을 고려하여 내용을 추가 해야 합니다.  -  .bashrc 또는.zshrc 설정을 참고 할 것  

## Mac GUI

그렇다면 왜 맥 GUI 앱에서는 PATH를 올바르게 찾을 수 없는 현상이 발생할까요? 사실 이 글의 제목에서 처럼 Sourcetree에서 문제의 원인을 찾으려 했을때는 맥 환경이 그렇다는 글들만 종종 발견할 수 있었고 다음과 같은 조치를 제안 하고 있었습니다. 

1. 터미널에서 open 명령으로 Sourcetreee 를 실행 할 것 
2. 마찬가지로 터미널에서 Sourcetree 의 CLI `stree` 명령으로 실행할 것

네, 확실히 잘 동작합니다. 하지만, 매번 터미널 명령으로 Sourcetreee를 실행하는 것은 불편합니다.
편리하게 사용 할 방법이 없을까 고민을 하다보니, 왜 Mac GUI 응용프로그램에서는 내가 알고있는 PATH 환경변수와 동일하지 않을까? 궁금해졌습니다

이와 관련된 아주 좋은 글을 발견하였고, 간단하게 번역을 해 두었습니다.  

1. [번역 - Mac OS에서 PATH 환경 변수 이해 하기](/posts/path-macos-best-practice-for-path-environment-variables-on-mac-os/)
2. [번역 - Mac OS GUI 앱에 대한 시스템 수준에서 PATH 환경 변수 설정 하기](/posts/instructs-mac-os-gui-apps-about-path-environment-variable/)

> Dock 이나 Spotlight을 통해 실행하는 Mac GUI 앱은 launchd 데몬을 통해 PATH를 상속 받습니다. 따라서 SHELL 에서 설정한 사용자 지정 PATH와 일치 하지 않습니다.  

## 조치

만약 맥의 GUI 앱에서도 SHELL에서 설정한 PATH 환경변수를 인식하기 위해서는 다음의 조치가 필요 합니다.

1. Husky 와 같이 연동되는 모듈의 설정 파일에서 설정
2. 터미널에서 앱을 실행 하기 
3. launchd 설정을 수정하여 쉘에 적용되는 설정을 상속

### 추가

시스템 전역 환경변수를 추가하는 방법은 macOS 10.10 이후부터 [변경](http://www.drjackyl.de/how/to/2017/08/15/Set_Global_Environment_Variables_in_macOS_10.10_and_later.html)된 것으로 파악되었습니다. 개인적으로는 Sourctree 외에는 Shell의 환경변수를 적용해야 하는 앱이 아직은 없기 때문에 이 방법을 테스트 하지는 않았습니다.
