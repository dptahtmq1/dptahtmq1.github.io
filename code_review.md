# [WIP] Code Review

# 개요

회사에서 한달 간 Code Review 활성화에 대한 교육을 듣고 있다. 이 교육에서 느낌 점을 간단하게 정리해 보고자 한다.

Code Review 활성화에는 크게 두 가지 방향이 있다. 첫 번째는 Soft Skill(문화 조성) 부분이고, 두 번째는 기술적  역량이다. Soft Skill은 리뷰어로서의 자세 및 마음가짐이라고 볼 수 있고, 기술적 역량은 리뷰를 더 쉽게 하기 위해서 필요한 것이라고 볼 수 있다.

### Code Review에 참여하지 않는 이유

1. 성과로 인정해 주지 않아서
2. 기술적인 역량이 부족해서
3. 시간이 부족해서
4. 변경 내용이 너무 많고, 코드를 알아보기 힘들어서

## Soft Skill(문화 조성)

코드 리뷰가 코드 품질을 높이기 위한 필수적인 활동이라는 마음가짐. 코드는 누군가의 소유물이 아닌 팀 전체의 산출물이라는 인식. 이러한 것들을 바탕으로 

## 기술적 역량?

두 번째 기술적 역량에 대해 다뤄보고자 한다. 이 부분은 리뷰어 / 리뷰이 모두에게 관련된다. 리뷰어는 지식이 부족해 Code Review에 어려움을 겪는 경우가 많고, 리뷰이는 리뷰어가 코드 리뷰를 더 쉽게 할 수 있도록 해야한다. 이 교육에서는 이러한 점을 해결하기 위해 아래 4가지 분야로 나누어서 정리한다.

- Clean Code
- TDD
- Refactoring
- Secure Coding

각각의 항목이 왜 Code Reivew와 연관 되는지에 대해 개인적인 생각을 정리해 보겠다.

### Clean Code

Clean Code는 Robert C. Martin의 책을 기준으로 한다. 개인적으로 Code Reivew에서 가장 중요한 부분이라고 생각한다. Clean Code야말로 리뷰어들이 리뷰를 쉽게 할 수 있게끔 도와주는 가장 큰 장치이다. 예를 들어 변수 명이 단순하게 x인 것과 numberOfApples 처럼 명확한 것이 있을 때 어떤 변수가 더 쉽게 받아들여질지는 명백하다. 리뷰어에게 코드 해석이 필요해지는 순간 리뷰를 포기하게 될 지도 모른다.

`int d; // elapsed time in days`

  ***vs***

`int elapsedTimeInDays;`

그리고 리뷰어도 Clean Code 원칙을 바탕으로 코드 리뷰에 쉽게 참여할 수 있다.  

### TDD

TDD가 Code Review와 왜 연관이 있는지? 처음에는 이해가 잘 되지 않았다. 여기서 테스트 코드의 가장 중요한 목적은 문서로서의 역할이다. 잘 짜여진 테스트 코드는 그 자체로 요구사항과 비지니스 로직을 나타낼 수 있다. 

여기서 TDD의 중요성이 나타난다. 요구사항을 바로 구현하는 것과 테스트 코드로 작성하는 것을 비교했을 때, 분명 테스트 코드가 요구사항을 더 명확하게 내포하게 된다. 테스트 코드를 먼저 작성 함으로써 요구사항에 대한 테스트 코드를 작성하게 되며, 이를 통해 문서로서의 가치를 높일 수 있다. 구현 → 테스트 코드 순으로 진행될 경우 구현한 비지니스 로직에 대한 테스트 코드가 작성되기 때문에 요구사항을 정확하게 표현하고 있지 않을 것이며, 리뷰어가 알아보기도 힘들게 된다.

### Refactoring

Refactoring은 Martin Fowler의 Refactoring 책을 기준으로 한다. 잘 알려진대로 Refactoring의 목적은 가독성과 유지 보수성이다. 여기서 Refactoring은 리뷰어가 리뷰를 쉽게 할 수 있도록 기반을 마련해주는 장치라고 유추할 수 있다. Refactoring을 통해 코드가 잘 정리되면 가독성이 증가하고, 이는 리뷰어의 참여를 촉구하는 기반 작업이 된다.

```
    if (anEmployee.seniority < 2) return 0;
    if (anEmployee.monthsDisabled > 12) return 0;
    if (anEmployee.isPartTime) return 0;
```
```
    if (isNotEligableForDisability()) return 0;
    
    function isNotEligableForDisability() {
        return ((anEmployee.seniority < 2)
                || (anEmployee.monthsDisabled > 12)
                || (anEmployee.isPartTime));
    }
```

또한 코드 리뷰의 기준이 될 수 있다. Refactoring 개념에 의거해 쉽게 코드 리뷰에 참여할 수 있다.

### Secure Coding

아직 안들었다.

## 번외

### Pair Programming

Pari Programming은 코드 리뷰 관점에서 봤을 때, 리뷰를 강제하는 제도이기 때문에 초반에 리뷰하는 습관을 들이는 데에 장점이 있다고 생각한다. 이는 기술적 역량 보다는 문화 조성에 도움을 줄 수 있는 내용일 듯 하다.
