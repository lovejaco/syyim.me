+++
title = "[스터디 정리] 하이퍼바이저의 종류"
date = "2020-06-05"
keywords = ["hypervisor"]
tags = [
"가상화",
"hypervisor"
]
+++

> 프론트엔드 팀을 위한 CI/CD 라는 주제의 사내 스터디를 진행하며 발표 내용을 정리/요약한 글이며 참고 링크의 글을 그대로 인용하였습니다.
> 스터디 중에 동료가 베어메탈에 대한 질문을 하였습니다. 이에 따로 정리한 내용입니다.

## 베어메탈

[ITWorld 용어풀이 | 베어메탈 - ITWorld Korea](http://www.itworld.co.kr/t/62077/vdi/90854)  
IT 업계에서 사용하는 베어메탈의 의미는 “어떤 소프트웨어도 담겨 있지 않은 하드웨어”를 의미하며, 다른 관점에서는 “하드웨어만을 구매할 수 있는 제품”을 의미하기도 합니다. [베어본 컴퓨터](http://www.itworld.co.kr/news/84523) 와 일부 유사한 의미를 갖고 있습니다.

물론 베어메탈이라는 용어는 이런 의미로 사용되고 있지만, 이 용어가 컴퓨팅 환경의 중요 용어로 부상한 것은 가상화 때문입니다. 가상화 아키텍처는 크게 두 가지로 분류하는데, 바로 호스트형 가상화와 베어메탈 가상화입니다. 말 그대로 호스트형 가상화는 하드웨어 상에 호스트 운영체제가 있고, 그 위에서 가상머신을 구현하는 방식입니다. 보통은 호스트 운영체제가 커널 수준에서 가상화 기술을 지원합니다.

반면에 베어메탈 가상화는 호스트 운영체제없이 하드웨어 상에 하이퍼바이저가 바로 설치되고, 이 위에 가상머신을 구현하는 것입니다. 이런 하이퍼바이저를 베어메탈 하이퍼바이저라고 부릅니다. 현재 서버용 하이퍼바이저의 대부분은 베어메탈 하이퍼바이저이며, 데스크톱용 하이퍼바이저의 다수가 호스트형 하이퍼바이저입니다.

![](https://cdn.shortpixel.ai/client/q_glossy,ret_img/https://www.pathpartnertech.com/wp-content/uploads/2020/02/hypervisors.png)

## 하이퍼바이저

[하이퍼바이저란?](https://www.redhat.com/ko/topics/virtualization/what-is-a-hypervisor)
하이퍼바이저는 [가상 머신(Virtual Machine, VM)](https://www.redhat.com/ko/topics/virtualization/what-is-a-virtual-machine) 을 생성하고 구동하는 소프트웨어입니다. 가상 머신 모니터(Virtual Machine Monitor, VMM)라고도 불리는 하이퍼바이저는 하이퍼바이저 운영 체제와 가상 머신의 리소스를 분리해 VM의 생성과 관리를 지원합니다.
하이퍼바이저로 사용되는 물리 하드웨어를 호스트라고 하며 리소스를 사용하는 여러 VM을 게스트라고 합니다.

하이퍼바이저는 CPU, 메모리, 스토리지 등의 리소스를 처리하는 풀로, 기존 게스트 간 또는 새로운 가상 머신에 쉽게 재배치할 수 있습니다.

### 하이퍼바이저 유형

가상화에 사용할 수 있는 하이퍼바이저에는 유형 1과 유형 2 하이퍼바이저가 있습니다.

#### 유형 1

![](https://devopsunleashed.files.wordpress.com/2017/05/type1.png)

네이티브 또는 베어 메탈 하이퍼바이저라고도 불리는 유형 1 하이퍼바이저는 호스트의 하드웨어에서 직접 구동되어 게스트 운영 체제를 관리합니다. 호스트 운영 체제 대신 VM 리소스는 하이퍼바이저에 의해 하드웨어에 직접 예약됩니다.
이러한 유형의 하이퍼바이저는 엔터프라이즈 데이터 센터와 서버 기반 환경에서 가장 일반적으로 사용됩니다.

#### 유형 2

![](https://devopsunleashed.files.wordpress.com/2017/05/type2.png)

호스트 하이퍼바이저라고도 불리는 유형 2 하이퍼바이저는 기존의 운영 체제에서 소프트웨어 레이어 또는 애플리케이션으로서 구동됩니다.

호스트 운영 체제에서 게스트 운영 체제를 추상화하는 방식으로 작동합니다. VM 리소스는 호스트 운영 체제에 따라 예약된 후 하드웨어에 대해 실행됩니다.

유형 2 하이퍼바이저는 개인 컴퓨터에서 여러 개의 운영 체제를 구동하려는 개인 사용자에게 이상적입니다.
VMware Workstation과 Oracle VirtualBox는 유형 2 하이퍼바이저의 예입니다.

## [추가 링크]

- [‘SW와 HW의 분리’··· 하이퍼바이저의 이해 - CIO Korea](http://www.ciokorea.com/news/36713)
- [IBM 기고 | 클라우드를 좀 더 쉽게, 베어메탈(Bare Metal) - CIO Korea](http://www.ciokorea.com/news/35402)
- [Are Containers Replacing Virtual Machines? - Docker Blog](https://www.docker.com/blog/containers-replacing-virtual-machines/)
- [Running Containers on Bare Metal vs. VMs: Performance and Benefits -](https://www.stratoscale.com/blog/data-center/running-containers-on-bare-metal/)
- [Future Clouds Could Be Just Containers On Bare Metal](https://www.nextplatform.com/2018/09/05/future-clouds-could-be-just-containers-on-bare-metal/)
