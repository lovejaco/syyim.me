+++
title = "[CI/CD] 개념 요약"
date = "2020-06-04"
description = "프론트엔드를 위한 CI/CD 스터디 정리"
tags = [
"CI/CD",
]
+++

> 프론트엔드 팀을 위한 CI/CD 라는 주제의 사내 스터디를 진행하며 발표 내용을 정리/요약한 글이며 참고 링크의 글을 그대로 인용하였습니다.

## What is CI / CD

CI/CD는 애플리케이션 개발 단계를 [자동화](https://www.redhat.com/ko/topics/automation/whats-it-automation) 하여 애플리케이션을 보다 짧은 주기로 고객에게 제공하는 방법입니다. CI/CD의 기본 개념은 지속적인 통합, 지속적인 서비스 제공, 지속적인 배포입니다. CI/CD는 새로운 코드 통합으로 인해 개발 및 운영팀에 발생하는 문제(일명 “ [인테그레이션 헬(integration hell)](https://www.solutionsiq.com/agile-glossary/integration-hell/) “)을 해결하기 위한 솔루션입니다.

### 지속적 통합(Continuous integration)

지속적인 통합을 수행하는 개발자는 변경 사항을 가능한 자주 main-line 브랜치로 병합합니다. 빌드를 작성하고 빌드에 대해 자동화된 테스트를 실행하여 개발자의 변경 사항을 검증합니다. 이렇게 하면 릴리스 일정에 따라 변경사항을 릴리스 브랜치에 병합하려 할 때 일반적으로 발생하는 통합 지옥을 피할 수 있습니다.

지속적인 통합은 새로운 커밋이 메인 브랜치에 통합될 때마다 애플리케이션이 중단되지 않는지 확인하기 위해 테스트 자동화에 중점을 둡니다.

### 지속적 전달 또는 제공(Continuous delivery)

[continuous-delivery](https://www.atlassian.com/continuous-delivery)는 지속적인 통합을 확장하여 고객에게 새로운 변경 사항을 지속 가능한 방식으로 신속하게 릴리스할 수 있도록 합니다. 개발자들이 애플리케이션에 적용한 변경 사항이 버그 테스트를 거쳐 리포지토리(예: [GitHub](https://redhatofficial.github.io/#!/main) 또는 컨테이너 레지스트리)에 자동으로 업로드되는 것을 뜻하며, 운영팀은 이 리포지토리에서 애플리케이션을 실시간 프로덕션 환경으로 배포할 수 있습니다. 이는 개발팀과 비즈니스팀 간의 가시성과 커뮤니케이션 부족 문제를 해결해 줍니다. 지속적인 제공은 최소한의 노력으로 새로운 코드를 배포하는 것을 목표로 합니다.

이론적으로 지속적으로 전달하면 매일, 매주, 격주로 또는 비즈니스 요구 사항에 맞는 것을 릴리스하기로 결정할 수 있습니다. 그러나 지속적인 제공의 이점을 실제로 얻으려면 가능한 한 빨리 프로덕션에 배포하여 문제 발생 시 문제 해결이 쉬운 작은 배치를 릴리스해야 합니다.

### 지속적 배포(Continuous deployment)

Continuous Deployment(또 다른 의미의 “CD”)는 Continuous delivery 보다 한 단계 더 발전합니다.
개발자의 변경 사항을 리포지토리에서 고객이 사용 가능한 프로덕션 환경까지 자동으로 릴리스하는 것을 의미합니다. 이는 애플리케이션 제공 속도를 저해하는 수동 프로세스로 인한 운영팀의 프로세스 과부하 문제를 해결합니다. 지속적인 배포는 파이프라인의 다음 단계를 자동화함으로써 지속적인 제공이 가진 장점을 활용합니다.

지속적인 배포는 더 이상 **출시일** 이 없으므로 고객과의 피드백 루프를 가속화하고 팀을 압박할 수 있는 훌륭한 방법 입니다. 개발자는 소프트웨어 제작에 집중할 수 있으며, 작업을 마치고 몇 분 후에 작업이 진행되는 것을 볼 수 있습니다.

![](https://www.redhat.com/cms/managed-files/ci-cd-flow-desktop_1.png)

> CI/CD는 지속적 통합 및 지속적 제공의 구축 사례만을 지칭할 때도 있고, 지속적 통합, 지속적 제공, 지속적 배포라는 3가지 구축 사례 모두를 의미하는 것일 수도 있습니다. 좀 더 복잡하게 설명하면 “지속적인 서비스 제공”은 때로 지속적인 배포의 과정까지 포함하는 방식으로 사용되기도 합니다.

### [Integration Hell](http://c2.com/xp/IntegrationHell.html)

현대적인 애플리케이션 개발에서는 여러 개발자들이 동일한 애플리케이션의 각기 다른 기능을 동시에 작업할 수 있도록 하는 것을 목표로 합니다. 그러나 특정한 날(“ [병합(머지) 하는 날(merge day)](https://thedailywtf.com/articles/Happy_Merge_Day!) “)을 정해 모든 분기 소스 코드를 병합하는 경우, 결과적으로 반복적인 수작업에 많은 시간을 소모하게 됩니다. 이렇게 하는 이유는 개발자가 애플리케이션에 변경 사항을 적용할 때 다른 개발자가 적용하는 변경 사항과 충돌할 가능성이 있기 때문입니다. 각 개발자가 각자의 로컬 IDE를 커스터마이징하는 경우 더욱 복합적인 문제가 될 수 있습니다.

[참고 링크]

- [Continuous integration vs. continuous delivery vs. continuous deployment](https://www.atlassian.com/continuous-delivery/principles/continuous-integration-vs-delivery-vs-deployment)
- [CI/CD(지속적 통합/지속적 제공): 개념, 방법, 장점, 구현 과정](https://www.redhat.com/ko/topics/devops/what-is-ci-cd)
- [지속적 전달이란 무엇입니까? - Amazon Web Services](https://aws.amazon.com/ko/devops/continuous-delivery/)
