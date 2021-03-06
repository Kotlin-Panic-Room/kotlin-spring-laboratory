# Java Garbage Collection Basics
> [원글](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)을 읽고 번역한 글 입니다. 의역 / 오역이 있을 수 있습니다.

---

## Overview

### Purpose


이 튜토리얼에서는 가비지 컬렉션이 Hotspot JVM에서 작동하는 방법에 대한 기본 사항을 다룹니다. 가비지 수집기의 작동 방식을 배웠으면 Visual VM을 사용하여 가비지 수집 프로세스를 모니터링하는 방법에 대해 알아봅니다. 마지막으로 Java SE 7 Hotspot JVM에서 사용할 수 있는 가비지 수집기에 대해 알아봅니다.

### Time to Complete

대략 1 시간

### Introduction

이 OBE는 Java에서 Java Virtual Machine(JVM) GC(가비지 컬렉션)의 기본 사항을 다룹니다. OBE의 첫 번째 부분에서는 가비지 컬렉션 및 성능에 대한 소개와 함께 JVM에 대한 개요가 제공됩니다. 다음 학습자들은 가비지 컬렉션이 JVM 내에서 어떻게 작동하는지에 대한 단계별 가이드를 제공합니다. 다음으로 학습자가 Java JDK에서 제공하는 모니터링 도구 중 일부를 사용해 보고 가비지 컬렉션에 대해 배운 내용을 실천할 수 있는 실습 활동이 제공됩니다. 마지막으로 Hotspot JVM에서 사용할 수 있는 가비지 컬렉션 구성표 옵션에 대한 섹션이 제공됩니다.

### Hardware and Software Requirements

다음은 하드웨어 및 소프트웨어 요구 사항 목록입니다.

* Windows XP 이상, Mac OS X 또는 Linux를 실행하는 PC입니다. 실제 작업은 Windows 7에서 수행되며 일부 플랫폼에서 테스트되지 않았습니다. 그러나 OS X나 Linux에서는 모든 것이 제대로 작동해야 합니다. 또한 둘 이상의 코어를 가진 기계가 바람직합니다.
* Java 7 업데이트 7 이상
* 최신 Java 7 데모 및 샘플 Zip 파일

--- 

## Java Technology and the JVM

자바는 프로그래밍 언어이며 1995년에 처음 발표 되었습니다. 유틸리티, 게임, 비즈니스 애플리케이션을 포함한 Java 프로그램에 힘을 실어주는 기본 기술입니다. 
자바는 전세계 8억 5천만 대 이상의 개인용 컴퓨터와 모바일 및 TV 장치를 포함한 전 세계 수십억 대의 장치에서 실행됩니다. 자바는 전체적으로 자바 플랫폼을 만드는 수많은 주요 구성 요소들로 구성되어 있습니다.

## _Java Overview_

### Java Runtime Edition

JRE는 자바 가상 머신(JVM), 자바 플랫폼 코어 클래스, 지원 자바 플랫폼 라이브러리로 구성됩니다. 컴퓨터에서 Java 애플리케이션을 실행하려면 세 가지 모두 필요합니다. 자바 7에서 자바 애플리케이션은 운영 체제의 데스크톱 애플리케이션으로 실행되지만 자바 웹 시작을 사용하여 웹에서 설치되거나 브라우저의 웹 임베디드 애플리케이션으로 실행됩니다.


### Java Programming Language

자바는 객체 지향 프로그래밍 언어로 다음과 같은 특징으 같습니다.

* 플랫폼 독립적 - 자바 애플리케이션은 클래스 파일에 저장되고 JVM에 로드되는 바이트코드로 컴파일됩니다. 애플리케이션은 JVM에서 실행되므로 다양한 운영 체제와 장치에서 실행할 수 있습니다.
* 객체 지향 - 자바는 C와 C++의 많은 특징들을 취하고 그것들을 개선시키는 객체 지향 언어입니다.
* 자동 가비지 컬렉션 - 자바는 자동으로 메모리를 할당하고 할당 해제하여 프로그램이 해당 작업에 부담을 주지 않도록 합니다.
* 풍부한 표준 라이브러리 - 자바에는 입출력, 네트워킹, 날짜 조작 등의 작업을 수행하는 데 사용할 수 있는 수많은 미리 만들어진 오브젝트가 포함되어 있다.


### Java Development Kit

자바 개발 키트(JDK)는 자바 애플리케이션을 개발하기 위한 도구 모음입니다. JDK를 사용하면 Java 프로그래밍 언어로 작성된 프로그램을 컴파일하여 JVM에서 실행할 수 있습니다. 또한 JDK는 응용 프로그램을 패키징하고 배포하는 도구를 제공합니다.

JDK와 JRE는 자바 API를 공유합니다. 자바 API는 개발자들이 자바 애플리케이션을 만들기 위해 사용하는 미리 패키지된 라이브러리 모음입니다. 자바 API는 문자열 조작, 날짜/시간 처리, 네트워킹, 데이터 구조(예: list, map, stack, queue)를 포함한 많은 일반적인 프로그래밍 작업을 완료하는 도구를 제공함으로써 개발을 더 쉽게 만듭니다.

### Java Virtual Machine

자바 가상 머신(, JVM)은 추상 컴퓨팅 머신입니다. JVM은 실행되도록 작성된 프로그램에 대해 머신처럼 보이는 프로그램입니다. 이러한 방식으로 자바 프로그램은 동일한 인터페이스와 라이브러리에 작성됩니다. 특정 운영 체제에 대한 각 JVM 구현은 자바 프로그래밍 명령을 로컬 운영 체제에서 실행되는 명령어와 명령어로 변환합니다. 이러한 방식으로 자바 프로그램은 플랫폼 독립성을 달성합니다.

썬 마이크로시스템즈에서 실행된 자바 가상 머신의 첫 번째 프로토타입 구현은 현대의 PDA(Personal Digital Assistant)와 유사한 휴대용 장치에 의해 호스팅되는 소프트웨어에서 자바 가상 머신 명령 세트를 에뮬레이트했습니다. Oracle의 현재 구현은 모바일, 데스크톱 및 서버 장치에서 Java 가상 머신을 에뮬레이트하지만 Java 가상 머신은 특정 구현 기술, 호스트 하드웨어 또는 호스트 운영 체제를 가정하지 않습니다. 본질적으로 해석되지는 않지만 실리콘 CPU의 명령어 집합을 컴파일하여 구현할 수 있니다다. 마이크로코드로 구현될 수도 있고 실리콘에 직접 구현될 수도 있습니다.

자바 가상 머신은 자바 프로그래밍 언어를 전혀 알지 못하며, 특정 이진 형식인 클래스 파일 형식만 알고 있습니다. 클래스 파일에는 Java 가상 머신 명령(또는 바이트 코드)과 기호 테이블 및 기타 보조 정보가 포함되어 있습니다.

보안을 위해 자바 가상 머신은 클래스 파일의 코드에 강력한 구문 및 구조적 제약을 가합니다. 그러나 유효한 클래스 파일로 표현될 수 있는 기능을 가진 모든 언어는 자바 가상 머신에 의해 호스팅될 수 있습니다. 일반적으로 이용 가능한 머신 독립 플랫폼에 매료되어 다른 언어의 구현자들은 자바 가상 머신에 의존할 수 있씁니다.


## _Exploring the JVM Architecture_


### Hotspot Architecture

HotSpot JVM은 강력한 기능 기반을 지원하고 고성능과 대규모 확장성을 실현할 수 있는 기능을 지원하는 아키텍처를 보유하고 있습니다. 예를 들어 HotSpot JVM JIT 컴파일러는 동적 최적화를 생성합니다. 다시 말해, 자바 애플리케이션이 실행되는 동안 최적화 결정을 내리고 기본 시스템 아키텍처를 대상으로 하는 고성능 네이티브 머신 명령을 생성합니다. 또한 HotSpot JVM은 런타임 환경과 멀티 스레드 가비지 콜렉터의 꾸준한 변화 및 지속적인 엔지니어링을 통해 사용 가능한 가장 큰 컴퓨터 시스템에서도 높은 확장성을 제공합니다.

![](https://velog.velcdn.com/images/roo333/post/dd0274cf-e5ab-493e-8aab-7a88188dea25/image.PNG)

JVM의 주요 구성 요소로는 클래스 로더, 런타임 데이터 영역, Execution Engine이 있습니다.


### Key Hotspot Components

다음 이미지에는 성능과 관련된 JVM의 주요 구성 요소가 강조 표시되어 있습니다.

![](https://velog.velcdn.com/images/roo333/post/cf5594a9-1cb2-4c39-8b45-66e30d529ec9/image.PNG)

JVM에는 성능 조정 시 중점을 두는 세 가지 구성 요소가 있습니다. 힙은 개체 데이터가 저장되는 위치입니다. 이 영역은 시작 시 선택한 가비지 수집기에 의해 관리됩니다. 대부분의 조정 옵션은 힙 크기를 조정하고 상황에 가장 적합한 가비지 수집기를 선택하는 것과 관련이 있습니다. 또한 JIT 컴파일러는 성능에 큰 영향을 미치지만 새로운 버전의 JVM과 튜닝할 필요가 거의 없습니다.


## _Performance Basics_

일반적으로 Java 애플리케이션을 튜닝할 때는 응답성 또는 처리량이라는 두 가지 주요 목표 중 하나에 초점을 맞춥니다. 튜토리얼이 진행되는 동안 이러한 개념을 다시 참조할 것입니다.


### 응답성


응답성은 애플리케이션 또는 시스템이 요청된 데이터 조각으로 얼마나 빨리 응답하는지를 나타냅니다. 그 예는 다음과 같습니다.

* 데스크톱 UI가 이벤트에 응답하는 속도
* 웹 사이트에서 페이지를 반환하는 속도
* 데이터베이스 쿼리가 반환되는 속도

응답성에 중점을 두는 애플리케이션의 경우, 큰 일시 중지 시간은 허용되지 않습니다. 단기간에 대응하는 데 초점이 맞춰져 있습니다.

### 처리량

처리량은 특정 기간 동안 응용프로그램에 의한 작업량을 최대화하는 데 초점을 맞춥니다. 처리량을 측정하는 방법의 예는 다음과 같습니다.

* 지정된 시간 동안 완료된 트랜잭션 수입니다.
* 배치 프로그램이 한 시간에 완료할 수 있는 작업 수입니다.
* 한 시간에 완료할 수 있는 데이터베이스 쿼리 수입니다.

처리량에 중점을 두는 애플리케이션의 경우 높은 일시 중지 시간이 허용됩니다. 높은 처리량 애플리케이션은 장기간 벤치마크에 집중하기 때문에 빠른 응답 시간은 고려 사항이 아닙니다.


---


## Describing Garbage Collection

> 자동 가비지 수집이란?

자동 가비지 수집은 힙 메모리를 살펴보고, 사용 중인 개체와 사용하지 않는 개체를 식별하고, 사용하지 않는 개체를 삭제하는 프로세스입니다. 사용 중인 개체 또는 참조된 개체는 프로그램의 일부가 여전히 해당 개체에 대한 포인터를 유지한다는 것을 의미합니다. 사용하지 않는 개체 또는 참조되지 않은 개체는 프로그램의 어떤 부분에서도 더 이상 참조되지 않습니다. 따라서 참조되지 않은 개체가 사용하는 메모리를 회수할 수 있습니다.

C와 같은 프로그래밍 언어에서 메모리를 할당하고 할당 해제하는 것은 수동 프로세스입니다. 자바에서 메모리 할당 해제 프로세스는 가비지 수집기에 의해 자동으로 처리됩니다. 기본 프로세스는 다음과 같이 설명할 수 있습니다.


### Step 1: Marking

이 과정의 첫 번째 단계는 Marking이라고 불립니다. 여기서 가비지 수집기가 사용 중인 메모리와 사용되지 않는 메모리를 식별합니다.

![](https://velog.velcdn.com/images/roo333/post/b40c4549-4e99-4368-9126-13b0749d5d83/image.PNG)

참조된 개체는 파란색으로 표시됩니다. 참조되지 않은 객체는 금색으로 표시됩니다. 모든 객체는 표시 단계에서 스캔되어 이 결정을 내립니다. 시스템의 모든 개체를 검색해야 하는 경우 이 프로세스는 시간이 많이 걸릴 수 있습니다.

### Step 2: Normal Deletion

일반 삭제는 참조되지 않은 개체를 제거하여 참조된 개체와 포인터를 여유 공간으로 남깁니다.

![](https://velog.velcdn.com/images/roo333/post/55ee8349-2881-419c-99ee-507dfa23f5a0/image.PNG)

Memory allocator는 새 개체를 할당할 수 있는 여유 공간 블록에 대한 참조를 보유합니다.

### Step 2a: Deletion with Compacting


성능을 더욱 향상시키기 위해 참조되지 않은 개체를 삭제할 뿐만 아니라 나머지 참조된 개체를 압축할 수도 있습니다. 참조된 객체를 함께 이동함으로써 새로운 메모리 할당을 훨씬 쉽고 빠르게 할 수 있습니다.

![](https://velog.velcdn.com/images/roo333/post/912bf0b3-3463-4c12-9ef1-8ddfd03ebcef/image.PNG)

### Why Generational Garbage Collection?

앞에서 설명한 바와 같이 JVM의 모든 개체를 표시하고 압축해야 하는 것은 비효율적입니다. 점점 더 많은 개체가 할당됨에 따라 개체 목록이 증가하고 증가하여 가비지 수집 시간이 점점 길어집니다. 그러나 응용 프로그램에 대한 경험적 분석을 통해 대부분의 개체는 수명이 짧다는 것을 알 수 있습니다.

다음은 그러한 데이터의 예입니다. Y축은 할당된 바이트 수를 나타내고 X 액세스는 시간 경과에 따라 할당된 바이트 수를 나타냅니다.

![](https://velog.velcdn.com/images/roo333/post/d4279b66-dab8-4f05-bbde-567a4221ee0f/image.gif)

보시다시피 시간이 지남에 따라 할당된 개체가 점점 더 적어집니다. 사실 대부분의 개체는 그래프의 왼쪽에서 더 높은 값으로 보여지듯이 수명이 매우 짧습니다.


### JVM Generations

객체 할당 동작에서 학습된 정보는 JVM의 성능을 향상시키는 데 사용될 수 있다. 따라서, 더미는 더 작은 부분이나 세대로 나뉩니다. 힙 파트는 Young 세대, Old 세대 또는 Tenured 세대와 Permanent 세대입니다.(* 자바 7 기준입니다.)

![](https://velog.velcdn.com/images/roo333/post/23f95213-755a-468f-b128-a88529a55f0b/image.PNG)

Young 세대는 모든 새로운 오브젝트가 할당되고 에이징되는 곳입니다. Young 세대가 채워지면, Minor GC가 수행됩니다. 마이너 컬렉션은 높은 객체 사망률을 가정하여 최적화될 수 있습니다. 죽은 오브젝트들로 가득 찬 Young 세대는 매우 빠르게 수집됩니다. 살아남은 일부 오브젝트는 에이징되어 구세대로 이동합니다.

**Stop the World Event** - 모든 Minor 가비지 컬렉션은 "Stop the World Event" 이벤트입니다. 즉, 작업이 완료될 때까지 모든 응용 프로그램 스레드가 중지됩니다. 

Old 세대는 오래 생존한 오브젝트를 저장하는 데 사용됩니다. 일반적으로 Young 세대 개체에 대한 임계값이 설정되고 해당 기간이 충족되면 개체가 다음 세대로 이동됩니다. 결국 Old 세대를 한번 더 가비지 컬렉팅 해야 합니다. 이 이벤트는 Major 가비지 컬렉션이라고 불립니다.

Major 가비지 컬렉션 또한 Stop the World 이벤트입니다. 종종 Major 가비지 컬렉션은 모든 살아있는 오브젝트를 포함하기 때문에 훨씬 더 느립니다. 따라서 응답 응용 프로그램의 경우 주요 가비지 컬렉션을 최소화해야 합니다. 또한 주요 가비지 컬렉션에 대한 Stop the World 이벤트의 길이는 이전 세대 공간에 사용되는 가비지 컬렉션의 종류에 영향을 받습니다.

Permanent 세대에는 응용 프로그램에서 사용되는 클래스 및 메서드를 설명하는 데 JVM에 필요한 메타데이터가 포함되어 있습니다. Permanent 세대는 응용 프로그램에서 사용 중인 클래스를 기반으로 런타임에 JVM에 의해 채워집니다. 또한 Java SE 라이브러리 클래스 및 메서드가 여기에 저장될 수 있습니다.

클래스는 JVM이 더 이상 필요하지 않으며 다른 클래스를 위한 공간이 필요할 경우 수집(언로드)될 수 있습니다. Permanent 세대는 full 가비지 컬렉션에 포함됩니다.

---

## The Generational Garbage Collection Process

이제 힙이 왜 다른 세대로 나뉘어져 있는지 이해하셨으니 이제 이러한 공간들이 정확히 어떻게 상호작용하는지 살펴보도록 하겠습니다. 다음 그림은 JVM에서 개체 할당 및 에이징 프로세스를 설명합니다.

![](https://velog.velcdn.com/images/roo333/post/7146ee3d-f9ed-4b26-9d55-b98d2aca9697/image.PNG)

1. 첫째, 모든 새 객체가 에덴 공간에 할당됩니다. 두 Survivor 공간은 모두 비어 있습니다.

![](https://velog.velcdn.com/images/roo333/post/6f21d3b8-4c80-44a7-8f65-91b5ac16b85f/image.PNG)

2. 에덴 공간이 가득 차면 Minor 가비지 컬렉션이 트리거됩니다.

![](https://velog.velcdn.com/images/roo333/post/ad478c99-971c-4cac-83f8-6bb4247e08bb/image.PNG)

3. 참조된 객체가 첫 번째 생존자 공간으로 이동됩니다. 참조되지 않은 개체는 에덴 공간을 지우면 삭제됩니다.

![](https://velog.velcdn.com/images/roo333/post/3b5e806f-c8a2-40e7-9cef-781969f8544d/image.PNG)

4. 다음 마이너 GC에서 에덴 공간에서도 같은 일이 일어납니다. 참조되지 않은 객체는 삭제되고 참조된 객체는 다음 Survivor 공간으로 이동됩니다(S1). 또한 첫 번째 생존 공간(S0)에 있는 마지막 마이너 GC의 객체는 나이를 증가시키고 S1로 이동됩니다. 모든 생존 개체가 S1로 이동되면 S0과 에덴이 모두 삭제됩니다. 이제 생존자 공간에서 서로 다른 나이가 든 오브젝트를 볼 수 있습니다.

![](https://velog.velcdn.com/images/roo333/post/e1cbfaf1-3419-4839-bd26-0637f4df87d0/image.PNG)

5. 다음 마이너 GC에서 동일한 프로세스가 반복됩니다. 그러나 이번에는 생존자 공간이 바뀝니다. 참조된 개체가 S0으로 이동됩니다. 살아남은 개체는 노후화됩니다. 에덴과 S1이 삭제되었습니다.


![](https://velog.velcdn.com/images/roo333/post/e8b0c42f-5f6a-4027-a9b8-067b0ff4af91/image.PNG)


6. 이 슬라이드는 프로모션을 보여 줍니다. 사소한 GC 후 오래된 객체가 특정 연령 임계값(이 예에서는 8)에 도달하면 젊은 세대에서 오래된 세대로 승격됩니다.

![](https://velog.velcdn.com/images/roo333/post/d3335d84-a0a9-4ce1-b8fc-3cf50e1ff48c/image.PNG)


7. 마이너 GC가 계속 발생함에 따라 객체는 Old 세대 공간으로 계속 승격됩니다.

![](https://velog.velcdn.com/images/roo333/post/19d3a655-caca-4867-beeb-d3f31ae27a05/image.PNG)

8. 그래서 Young 세대들의 모든 과정을 망라하고 있습니다. 결국, 그 공간을 청소하고 압축하는 Major GC가 Old 세대에게 수행될 것입니다.


---

## Performing Your Own Observations

![](https://velog.velcdn.com/images/roo333/post/8995e686-826d-4b0b-9c93-4e363a53a1d4/image.png)

>VisualVM을 통해 실습해보고 싶은 신 분은 상단에 원글 링크에서 실습해보세요!

---

## Java Garbage Collectors

이제 가비지 수집의 기본 사항에 대해 알게 되었고 샘플 애플리케이션에서 가비지 수집기가 작동하는 것을 관찰했습니다. 이 섹션에서는 Java에 사용할 수 있는 가비지 수집기 및 가비지 수집기를 선택하는 데 필요한 명령줄 스위치에 대해 알아봅니다.

### Serial GC
Serial Collection은 Java SE 5 및 6의 클라이언트 스타일 시스템에 대한 기본값입니다. Serial Collection를 사용하면 부 및 주요 가비지 수집이 모두 직렬로 수행됩니다(단일 가상 CPU 사용). 또한 mark-compact 수집 방식을 사용합니다. 이 방법은 이전 메모리를 힙의 시작 부분으로 이동하여 새 메모리 할당이 힙 끝에서 단일 연속 메모리 청크로 만들어집니다. 이렇게 메모리를 압축하면 새 메모리 청크를 힙에 더 빠르게 할당할 수 있습니다.

### 사용 사례
Serial GC는 일시 중지 시간 요구 사항이 낮지 않고 클라이언트 스타일 시스템에서 실행되는 대부분의 응용 프로그램에서 선택하는 가비지 수집기입니다. 가비지 수집 작업을 위해 단일 가상 프로세서(따라서 이름)만 활용합니다. 그러나 오늘날의 하드웨어에서 직렬 GC는 비교적 짧은 최악의 경우 일시 중지(전체 가비지 수집의 경우 약 몇 초)와 함께 수백 MB의 Java 힙으로 많은 중요하지 않은 응용 프로그램을 효율적으로 관리할 수 있습니다.

직렬 GC의 또 다른 인기 있는 용도는 동일한 시스템에서 많은 수의 JVM이 실행되는 환경입니다(경우에 따라 사용 가능한 프로세서보다 많은 JVM이 있습니다!). JVM이 가비지 수집을 수행할 때 이러한 환경에서는 가비지 수집이 더 오래 지속되더라도 나머지 JVM에 대한 간섭을 최소화하기 위해 하나의 프로세서만 사용하는 것이 좋습니다. 그리고 Serial GC는 이 트레이드 오프에 잘 맞습니다.

마지막으로, 최소한의 메모리와 몇 개의 코어를 가진 임베디드 하드웨어의 확산으로 Serial GC가 다시 돌아올 수 있습니다.

### 명령줄 스위치
직렬 수집기를 활성화하려면 다음을 사용하십시오.
-XX:+UseSerialGC

다음은 Java2Demo를 시작하기 위한 샘플 명령줄입니다.
자바 -Xmx12m -Xms3m -Xmn1m -XX:PermSize=20m -XX:MaxPermSize=20m -XX:+UseSerialGC -jar c:\javademos\demo\jfc\Java2D\Java2demo.jar

### Parallel GC
병렬 가비지 수집기는 여러 스레드를 사용하여 Young 세대 가비지 수집을 수행합니다. 기본적으로 N개의 CPU가 있는 호스트에서 병렬 가비지 수집기는 컬렉션에서 N개의 가비지 수집기 스레드를 사용합니다. 가비지 수집기 스레드 수는 명령줄 옵션으로 제어할 수 있습니다.
-XX:ParallelGCThreads=<원하는 번호>

단일 CPU가 있는 호스트에서는 Parallel 가비지 수집기가 요청된 경우에도 기본 가비지 수집기가 사용됩니다. 2개의 CPU가 있는 호스트에서 병렬 가비지 수집기는 일반적으로 기본 가비지 수집기만큼 성능을 발휘하며 2개 이상의 CPU가 있는 호스트에서는 Young 세대의 가비지 수집기 일시 중지 시간을 줄일 수 있습니다. Parallel  GC는 두 가지 형태로 제공됩니다.

### 사용 사례
병렬 수집기는 처리량 수집기라고도 합니다. 여러 CPU를 사용하여 애플리케이션 처리 속도를 높일 수 있기 때문입니다. 이 수집기는 많은 작업을 수행해야 하고 긴 일시 중지가 허용되는 경우에 사용해야 합니다. 예를 들어 보고서 또는 청구서 인쇄 또는 많은 수의 데이터베이스 쿼리 수행과 같은 일괄 처리.

### -XX:+UseParallelGC
이 명령줄 옵션을 사용하면 단일 스레드 구세대 수집기가 있는 다중 스레드 젊은 세대 수집기를 얻을 수 있습니다. 이 옵션은 또한 이전 세대의 단일 스레드 압축을 수행합니다.

다음은 Java2Demo를 시작하기 위한 샘플 명령줄입니다.
자바 -Xmx12m -Xms3m -Xmn1m -XX:PermSize=20m -XX:MaxPermSize=20m -XX:+UseParallelGC -jar c:\javademos\demo\jfc\Java2D\Java2demo.jar

### -XX:+UseParallelOldGC
-XX:+UseParallelOldGC 옵션을 사용하면 GC는 다중 스레드 Young 세대 수집기이자 다중 스레드 이전 세대 수집기입니다. 또한 다중 스레드 압축 수집기입니다. HotSpot은 Old 세대에서만 압축을 수행합니다. HotSpot의 Young 세대는 카피 컬렉터로 간주됩니다. 따라서 압축할 필요가 없습니다.

압축은 개체 사이에 구멍이 없는 방식으로 개체를 이동하는 행위를 설명합니다. 가비지 수집 스윕 후 라이브 개체 사이에 구멍이 남을 수 있습니다. 압축은 남아 있는 구멍이 없도록 개체를 이동합니다. 가비지 수집기가 압축되지 않은 수집기일 수 있습니다. 따라서 Parallel GC와 Parallel 압축 GC의 차이점은 후자가 가비지 수집 스윕 후 공간을 압축한다는 것입니다. 전자는 그렇지 않습니다.

다음은 Java2Demo를 시작하기 위한 샘플 명령줄입니다.
자바 -Xmx12m -Xms3m -Xmn1m -XX:PermSize=20m -XX:MaxPermSize=20m -XX:+UseParallelOldGC -jar c:\javademos\demo\jfc\Java2D\Java2demo.jar

### 동시 마크 스윕(CMS) 수집기
Concurrent Mark Sweep(CMS) 수집기(동시 낮은 일시 중지 수집기라고도 함)는 Permanent 세대를 수집합니다. 대부분의 가비지 수집 작업을 응용 프로그램 스레드와 동시에 수행하여 가비지 수집으로 인한 일시 중지를 최소화하려고 합니다. 일반적으로 동시 낮은 일시 중지 수집기는 라이브 개체를 복사하거나 압축하지 않습니다. 가비지 수집은 라이브 개체를 이동하지 않고 수행됩니다. 조각화가 문제가 되면 더 큰 힙을 할당하십시오.

참고: Young 세대의 CMS 수집기는 동일한 알고리즘을 사용합니다.
