# Garbage Collection

참고자료

- [https://perfectacle.github.io/2019/05/11/jvm-gc-advanced/](https://perfectacle.github.io/2019/05/11/jvm-gc-advanced/)
- [https://perfectacle.github.io/2019/05/07/jvm-gc-basic/#Mark-and-Sweep-Algorithm](https://perfectacle.github.io/2019/05/07/jvm-gc-basic/#Mark-and-Sweep-Algorithm)

GC 원칙

1. 알고리즘은 반드시 모든 가비지를 수집해야 한다.
2. 살아 있는 객체는 절대로 수집해선 안된다.

## Mark and sweep

전체적인 GC 알고리즘

1. 할당 리스트(allocated list)를 순회하면서 마크 비트(mark bit)를 지운다.
2. GC 루트부터 살아 있는 객체를 찾는다.
3. 이렇게 찾은 객체마다 마크 비트를 세팅한다.
4. 할당 리스트를 순회하면서 마크 비트가 세팅되지 않은 객체를 찾는다.
    1. 힙에서 메모리를 회수해 프리 리스트(free list)에 되돌린다.
    2. 할당 리스트에서 객체를 삭제한다.

![Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled.png](Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled.png)

### 객체를 어떻게 찾는가?

DFS(depth first search) 방식으로 찾습니다. 이렇게 해서 생성된 객체 그래프를 live object graph라고 합니다.

heap 상태를 보는 명령어 : `jmap -histo`

### 관련 용어

STW : Stop the world. GC가 발생하는 동안에는 모든 애플리케이션 스레드가 중단됩니다.

동시 : GC 스레드는 애플리케이션 스레드와 동시 실행될 수 없습니다.

병렬 : 여러 스레드를 동원해서 가비지 수집을 합니다. (thread dump를 보면 GC thread들이 다수 존재합니다.)

압착(compaction) : 살아남은 객체들은 GC 사이클 마지막에 연속된 단일 영역으로 배열됩니다. 이는 메모리 단편화(memory fragmentation)을 방지합니다.

GC 루트 및 아레나

GC 루트는 메모리의 고정점(anchor point)로 메모리 풀 외부에서 내부를 가리키는 포인터입니다. 종류는 아래와 같습니다.

- stack frame
- JNI
- 레지스터(호이스트된 변수)
- 전역 객체
- 로드된 클래스의 메타데이터

할당과 수명

자바의 가비지 수집 프로세스는 보통 유입된 메모리 할당 요청을 수용하기에 메모리가 부족할 때 작동하여 필요한 만큼 메모리를 공급합니다. 자바에서 GC가 일어나는 주된 원인은 다음 두 가지 입니다.

1. 할당률 : 일정 기간 동안 새로 생성된 객체가 사용한 메모리량(MB/s)
2. 객체 수명 : 예측하기가 힘듦

### 약한 세대별 가설(Weak Generational Hypothesis)

1. JVM에서 거의 대부분의 객체는 아주 짧은 시간만 살아 있지만, 나머지 객체는 기대 수명이 훨씬 길다.

    → 단명 객체를 쉽고 빠르게 수집할 수 있게 설계해야 하며, 장수 객체와 단명 객체를 완전히 떼어놓는게 가장 좋다.

2. 늙은 객체가 젊은 객체를 참조할 일은 거의 없다.

핫스팟은 약한 세대별 가설을 십분 활용합니다.

1. 객체마다 세대 카운트(객체가 지금까지 무사 통과한 GC 횟수)를 센다.
2. 큰 객체를 제외한 나머지 객체는 Eden 공간에 생성한다. 여기서 살아남은 객체는 다른 곳으로 옮긴다.
3. 충분히 오래 살아남은 객체들은 별도의 메모리 영역(Old 또는 Tenured)에 보관한다.

[그림 필요함. generational collection]

![Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%201.png](Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%201.png)

(외부에서 young 세대를 가리키는 포인터를 계속 추적하는 방법이 중요. card table이라는 자료구조를 통해 old → young 참조하는 정보를 기록함)

카드 테이블(card table) : old → young 참조하는 정보를 기록하는 테이블

스레드 로컬 할당

JVM은 에덴 영역을 여러 버퍼로 나누어 각 스레드가 새 객체를 할당하는 구역으로 활용하도록 합니다. 이 구역을 스레드 로컬 할당 버퍼(TLAB, Thread-Local Allocation Buffer)라고 합니다.

(애플리케이션 스레드가 할당된 버퍼를 다 채우면 JVM은 새 영역을 가리키는 포인터를 내어줍니다.)

반구형 수집기(hemispheric evacuating collector)

실제로 장수하지 못한 객체를 임시 수용수에 담아 둡니다. 핫스팟에서는 이 공간을 Survivor 영역이라고 합니다. 덕분에 단명 객체가 테뉴어드 세대를 어지럽히지 않고, 풀 GC 발생 빈도를 줄일 수 있습니다.

- 수집기가 라이브 반구를 수집할 때 객체들은 다른 반구로 압착시켜 옮기고, 수집된 반구는 비워서 재사용한다.
- 절반의 공간은 항상 완전히 비운다.

## Parallel Collector

Java 8 까지 default collector입니다.

![Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%202.png](Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%202.png)

### 영 세대 병렬 수집

스레드가 에덴 영역에 객체를 할당하려 하는데, 자신이 할당받은 TLAB 공간은 부족하고, JVM은 새 TLAB를 할당할 수 없을 때 영 세대 수집이 발생합니다.

영 세대 수집이 일어나면 JVM은 전체 애플리케이션 스레드를 중단 시킵니다.

1. 전체 애플리케이션 스레드가 중단되면 핫스팟은 영 세대(eden 및 비어있지 않은 survivor)를 뒤져서 가비지 아닌 객체를 골라냅니다.
2. 살아남은 객체를 현재 비어있는 survivor 영역으로 모두 방출한 후, 세대 카운트를 늘려 한 차례 이동했음을 기록합니다.
3. eden과 객체를 방출한 survivor 영역을 재사용 가능한 빈 공간으로 표시하고, 애플리케이션 스레드를 재시작합니다.

### 올드 세대 병렬 수집

영 세대와 달리 하나의 연속된 메모리 공간에서 압착하는 수집기 입니다. 올드 세대 내부에서 객체들을 재배치해서 늙은 객체가 죽고 빠져 버려진 공간을 회수합니다.

Mark - Sweep - Compaction 

### 병렬 수집기의 한계

1. STW 시간이 힙 크기에 거의 비례합니다.
    1. 영 수집 : 약한 세대별 가설에 따라 극소수 객체만 살아남아 있음
    2. 올드 수집 : 영역 내 살아 있는 객체 수만큼 마킹 시간도 늘어난다는 점. 올드 객체는 장수한 객체이므로 풀 수집 시 이들 중 상당수는 살아남을 공산이 큽니다.

샘플

> 전체 : 2GB
올드 : 1.5GB
영 : 500MB
에덴 : 400MB
S1 : 50MB
S2 : 50MB
할당률 : 100MB/s
영 GC 시간 : 2ms
풀 GC 시간 : 100ms
객체 수명 : 100ms

4초마다 영 GC 발생 → 마지막 100ms 에 생성된 애들은 Survivor 영역으로 이동 → 다음 번에 죽음

2초 간 정상 상태 할당 : 100MB/s

1초 간 할당률 급증 : 1GB/s

100초 후 정상 상태로 돌아옴 : 100MB/s

처음 2초 200메가 에덴 할당 → 0.2초만에 200메가 에덴에 추가 할당 → 나이가 100밀리초 이하인 객체가 100메가 → 살아남은 객체 용량이 서바이버 공간보다 큼 → 테뉴어드로 승격(조기 승격, premature promotion) 

→ 승격된 객체가 금새 죽음 → 테뉴어드 세대가 지저분해짐

GC 선정 시 고려해야 할 항목

- 중단 시간
- 처리율(애플리케이션 런타임 대비 GC 시간 %)
- 중단 빈도
- 회수 효율(GC 사이클 당 얼마나 많은 가비지가 수집 되는지)
- 중단 일관성

## 동시 GC 이론

GC 사이클은 어떤 고정된, 예측 가능한 일정에 맞춰 발생하는게 아니라, 순전히 그때그때 필요에 의해 발생합니다.

범용 GC는 중단 결정을 내리는데 참고할 만한 도메인 지식이 없습니다. 메모리 할당은 불확정성을 유발하는 직접적인 원인으로, 들쑥날쑥한 양상을 보이게 됩니다.

### JVM 세이프포인트

STW 가비지 수집을 하려면 애플리케이션 쓰레드를 모두 중단시켜야 합니다. 그래서 JVM은 스레드마다 세이프포인트라는 특별한 실행 지점을 둡니다. (GC 스레드가 OS에게 무조건 어플리케이션 스레드를 강제 중단시켜달라고 요청할 방법이 없기 때문)

세이프포인트 : 스레드의 내부 자료 구조가 훤히 보이는 지점으로, 여기서 어떤 작업을 하기 위해 스레드는 잠시 중단될 수 있습니다.

- JVM은 강제로 스레드를 세이프포인트 상태로 바꿀 수 없다.
- JVM은 스레드가 세이프포인트 상태에서 벗어나지 못하게 할 수 있다.

JVM이 세이프포인트 시간(time to safepoint) 플래그를 세팅하면, 각 애플리케이션 스레드는 폴링하면서 이 플래그를 확인합니다.

- Thread가 lock 대기 또는 wait 상태
- JNI 코드를 실행한다.

이와 관련 참조: [https://tv.kakao.com/channel/3693125/cliplink/414199823?metaObjectType=Channel](http://tv.kakao.com/v/414191712)

### 삼색 마킹(tri-color marking)

Mark and Sweep 알고리즘에서 2가지 색(마킹되었음 / 안되었음)을 쓴 것과 대비되며, 3가지 색을 써서 마킹하는 알고리즘 입니다. 애플리케이션이 멈추지 않으면서 GC를 위한 작업을 하기 위한 목적입니다.

1. 회색 : 아직 처리되지 않은 객체
2. 검은색 : 해당 객체가 참조하고 있는 객체를 모두 식별한 상태
3. 흰색 : 해당 객체를 참조하고 있는 객체가 없는 상태

![Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%203.png](Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%203.png)

- GC 루트를 회색 표시한다.
- 다른 객체는 모두 흰색 표시한다.
- 마킹 스레드가 회색 노드로 랜덤하게 이동한다.
- 이동한 노드를 검은색 표시하고, 이 노드가 가리키는 모든 흰색 노드를 회색 표시한다.
- 회색 노드가 하나도 남지 않을 때까지 위 과정을 되풀이한다.
- 검은색 : 살아남음, 흰색 : 수집 대상

동시 수집은 SATB(snapshot at the beginning) 기법을 활용합니다. 수집 사이클 이후에 할당된 객체를 라이브 객체로 간주합니다. 이로 인해 수집 도중 객체 그래프에 변화가 발생할 수 있습니다.

**G1은 SATB 알고리즘 사용** : Initial mark 때 heap 의 virtual snapshot 을 찍고, 그 시점에 살아 있던 객체는 남은 marking 단계 동안 살아 있다고 가정. 이 말은 marking 동안 해당 오브젝트가 죽을 수 있음. 이는 다른 collector 대비 제대로 메모리 수집 못한다는 의미?

그러나 이는 Remark 때 latency 이점이 있음.

객체가 변경됐을 때 재마킹하게끔 하거나(쓰기 배리어), 모든 변경 사항을 큐에 넣어 놓고 부차적인 fixup 단계에서 바로잡는 방법 등이 있습니다. 이는 GC마다 다릅니다.

## CMS(Concurrent Mark Sweep)

CMS는 중단 시간을 아주 짧게 하려고 설계된, 올드 영역 전용 수집기입니다. 보통 Parallel GC를 조금 변형한 ParNew와 함께 씁니다.

CMS는 중단 시간을 최소화하기 위해 어플리케이션 스레드 실행 중에 가급적 많은 일을 합니다. 기본적으로 가용 스레드 절반을 GC 동시 단계에 사용합니다.

삼색 마킹 알고리즘을 사용하기 때문에 GC와 함께 어플리케이션을 돌릴 수 있습니다.

1. Initial Mark(STW)
2. Concurrent Mark
3. Concurrent Preclean
4. Remark(STW)
5. Concurrent Sweep
6. Concurrent Reset

![Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%204.png](Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%204.png)

### 단점

1. 단일 풀 GC 사이클이 Parallel GC 보다 길다.
2. CMS GC 사이클이 실행되는 동안 어플리케이션 처리율은 감소한다.
3. GC 수행에 더 많은 CPU 시간과 메모리가 필요하다.(알고리즘도 복잡하고, 객체 추적 용..)
4. Compcation을 하지 않기 때문에 단편화가 발생할 수 있다.
5. CMS 도중 young GC가 발생하는 경우 코어 절반만 사용하기 때문에 Parallel 대비 더 느리다.

### CMF(Concurrent Mode Failure)

할당압이 너무 높은 경우 / 힙 단편화에 의해 old 영역에 공간이 부족한 사태가 발생할 수 있습니다. 이 경우에는 어쩔 수 없이 풀 STW를 유발하는 Parallel Old GC 방식으로 돌아갑니다.

## G1

Java 9부터 디폴트. 통계를 계산해가면서 GC 작업량을 조절한다.

특성

- CMS보다 훨씬 튜닝하기 쉽다.
- 조기 승격에 덜 취약하다.
- 대용량 힙에서 확장성이 우수하다.
- 풀 STW 수집을 없앨 수 있다.

참조 : [https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector.htm#JSGCT-GUID-C268549C-7D95-499C-9B24-A6670B44E49C](https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector.htm#JSGCT-GUID-C268549C-7D95-499C-9B24-A6670B44E49C)

**기본 컨셉**

G1은 일시 중지 시간 목표를 모니터링함. generational, incremental, parallel, mostly concurrent, stop-the-world

**레이아웃**

![Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%205.png](Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%205.png)

![Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%206.png](Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%206.png)

G1 힙은 region으로 구성됩니다. heap size / 2048 계산 후 2의 배수에 맞게 반올림하여 region의 개수가 정해집니다. region으로 나눔으로써 latency를 예측할 수 있게 됩니다. `maxTargetPauseTime` 값을 설정할 수 있으며, 심지어 garbage가 남게 되더라도 collection을 중단함으로써 이 값을 준수하도록 노력합니다.

**humongous region**

region 절반 이상을 점유한 객체는 거대 객체(humongous object)로 간주하여 별도 공간에 곧바로 할당됩니다. 얘네들은 old gen의 연속적인 region에 할당됨.(sequence의 마지막 region은 낭비됨)

Cleanup pause 또는 Full GC 때 reclaimed됨. 그러나 primitive type array의 경우는 어떤 모든 GC pause에서 처리되도록 되어 있음

humongous 객체를 할당하면 GC가 조기에 발생할 수 있음. G1은 모든 거대한 개체 할당에서 Heap Occupancy threshold를 확인하고 initial mark young collection을 바로 시작할 수 있음

RSet(remembered set) : 카드 테이블과 유사하며, 어떤 객체가 어느 region에 저장되어 있는지를 관리합니다. 덕분에 G1은 영역 내부를 바라보는 레퍼런스를 찾으려고 전체 힙을 다 뒤질 필요가 없습니다.

**IHOP(Initiating Heap Occupancy Percent)**

Initial Mark collection이 trigger되는 threshold로, old gen의 % 로 설정됨.

G1은 기본적으로 marking phase가 얼마나 걸리는지, marking cycle 동안 old gen에 얼만큼의 메모리가 할당되는지를 지켜보고 최적의 IHOP을 결정합니다. 이것을 Adaptive IHOP라고 부릅니다.

옵션을 통해 이런 동작을 끌 수 있고, threshold 값을 정할 수도 있음. 내부적으로 언제 mixed gc를 시작할지도 이를 통해 정하게 됨.

**GC Cycle**

![Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%207.png](Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%207.png)

1. Young-only phase
    1. young-only collection + old gen으로 승격. young-only 와 space-reclamation phase 사이의 전환은 old gen 점유율이 특정 threshold에 도달했을 때 발생. (IHOP) 이 경우에 일반적인(regular) young-only collection 대신 initial mark young-only collection을 시작함
    2. Initial mark : 일반적인 young-only collection 외에 marking process를 추가로 수행. 이 단계에서 old gen의 모든 reachable object를 결정(얘들은 space-reclamation 단계에서 살아남음). marking이 완전하게 끝나지 않는 동안, regular young collection이 발생. Marking은 STW 단계 Remark / Cleanup 로 끝납니다.
    3. Remark : 마킹을 종료하고, global reference processing 및 class unloading을 수행함. Remark와 cleanup 사이에 G1은 살아있는 오브젝트 정보를 동시에 계산하며, 이는 cleanup 때 내부 데이터 구조를 업데이트 하는데 사용합니다.
    4. Cleanup : region을 완전하게 비웁니다. 그리고 space-reclamation 단계가 필요한지 결정합니다. 
2. Space-reclamation phase : 공간 회수 단계
    1. mixed collections. young + old gen의 살아 있는 애들 대피. 이 단계는 G1이 old gen 영역을 더 대피 시켜봤자 충분한 공간을 확보할 가치가 없다고 판단할 때 종료됩니다. 즉 더 많이 확보할만한 공간이 없을 때
    2. Space-reclamation이 종료되면 다시 1번 young-only로 돌아간다.

알고리즘

- 동시 마킹 단계를 이용한다.
- 방출 수집기다.
- 통계적으로 압착한다.

GC 사이클이 한 번 돌 때마다 얼마나 많은 가비지를 수집할 수 있는지 그 수치를 보관합니다.

G1 단계

1. Initial Mark(**STW**) : old region 객체들이 참조하는 survivor region을 찾습니다.
2. Root Region Scan : Initial Mark에서 찾는 survivor region에 대한 GC 대상 객체 스캔 작업을 합니다.
3. Concurrent Mark : 전체 힙의 region에 대해 스캔 작업을 진행합니다.
4. Remark(**STW**) : 최종적으로 GC 대상에서 제외될 객체를 식별합니다.
5. Clean Up(**STW**) : 살아있는 객체가 가장 적은 region에 대해 수거를 진행합니다.
6. Copy : GC 대상 region이었지만 Clean Up 과정에서 완전히 비워지지 않은 region의 살아남은 객체들을 새로운 Region에 복사하여 Compaction 작업을 수행합니다.

![Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%208.png](Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%208.png)

Mixed Garbage Collections

Concurrent Mark 후에 young 영역 GC 시에 일부 old region을 함께 포함 시키는 Collector입니다. `XX:G1MixedGCLiveThresholdPercent=65` 옵션에 의해 설정됩니다.

**Evacuation failures case**

살아 있는 오브젝트가 많은 경우, evacuation이 실패할 수 있음. 즉 살아 있는 오브젝트를 옮길 공간이 부족한 경우. G1은 이 실패는 GC의 마지막 부분에서 발생한다고 가정하고, 대부분의 오브젝트가 이미 옮겨졌다고 간주하여 marking을 끝내고, space-reclamation 을 시작한다.

만약 이 가정이 유지되지 않으면 G1은 결국 Full GC를 수행하게 됨.

**Young-only phase Generation Sizing**

G1은 young only collection 끝날 때 항상 young gen의 크기를 측정함. 이를 통해 pause time 목표를 만족할 수 있음(`XX:MaxGCPauseTimeMillis`, `XX:PauseTimeIntervalMillis`) - 실제 pause time에 대한 장기적인 관찰을 기반으로. 비슷한 크기의 young gen을 evacuate 하는데 걸린 시간 고려.

G1은 `XX:G1NewSizePercent` and `-XX:G1MaxNewSizePercent` 사이에서 pause time을 만족할 수 있는 young gen size를 유동적으로 조절함.

**Space-Reclamation Phase Generation Sizing**

Spcae-reclamation 동안 G1은 GC pause 동안 회수된 old gen의 space를 최대화하려고 시도합니다. young gen은 허용된 최소치(XX:G1NewSizePercent)로 설정되며, old region은 pause time goal을 초과할 때까지 추가됨.

XX:G1MixedGCLiveThresholdPercent 보다 낮은 애들이 collection 대상이 됨. XX:G1MixedGCCountTarget 값에 의해 GC 당 얼마나 많은 old gen region이 선택될지 결정됨(live가 가장 작은 애들부터?). 그리고 이 phase는 컬렉션 집합 후보 region들에서 회수할 수 있는 남은 공간 양이 XX:G1HeapWastePercent. 보다 작으면 종료됨.

**Options**

- `XX:MaxGCPauseMillis=200` : 최대 일시 정지 시간 목표. default 200ms.

참조

- [https://b.luavis.kr/server/g1-gc](https://b.luavis.kr/server/g1-gc)
- [https://www.oracle.com/technical-resources/articles/java/g1gc.html](https://www.oracle.com/technical-resources/articles/java/g1gc.html)
- [https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector.htm#JSGCT-GUID-C268549C-7D95-499C-9B24-A6670B44E49C](https://docs.oracle.com/javase/9/gctuning/garbage-first-garbage-collector.htm#JSGCT-GUID-C268549C-7D95-499C-9B24-A6670B44E49C)

## Z GC

java 15에 production ready. 왜 탄생 했을까? → G1은 많이 개선 되었으나 STW 동안 모든 동작이 멈추는 경우는 여전히 존재한다. 

Linux에서는 JDK11부터 사용할 수 있음.

main goals

- GC pause time을 최대 10ms 이하로 줄인다.
- KB ~ large TB heap 을 모두 커버한다.
- G1 대비 15% 이하의 application throughput 감소
- Colored pointer와 load barrier를 통해 앞으로의 GC 기반 마련

ZGC는 STW 를 가능한 적게 유지하려 한다. heap size 증가한다고 pause time이 증가하지 않음. 그래서 large heap 을 사용하는 server application에 적합.

어떻게??

1. marking 단계에서 reachable object를 어떻게 저장할 것인가? reference state를 bit로 저장 : reference coloring
2. memory fragmentation을 줄이자 : relocation

이로 인한 문제점 : relocation 발생 후 thread가 해당 오브젝트를 이전 주소로 접근하려 할 때 → load barriers 를 통해 이를 해결 : remapping

### Marking

3 단계로 나눠짐. marking에 marked0, marked1 이라는 bit를 사용함.

1. **STW**. root reference를 찾고 mark함. root 탐색이라 굉장히 짧음.
2. concurrent. root reference부터 object graph 를 순회하고 도달하는 모든 오브젝트에 마킹함. 그리고 load barrier가 unmarked reference를 발견하면 그것 역시 mark
3. **STW**. weak reference와 같은 edge case 처리

### Reference Coloring

![Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%209.png](Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%209.png)

32 bit : 4 GB밖에 못 표현함. 그래서 ZGC는 64-bit reference 사용함. 즉 64bit platform 에서만 동작 가능하다.

- finalizable : 연결할 수 없는 object. garbage로 간주
- remap : reference가 최신인지 아닌지 나타냄.

### Relocation

파편화된거 정리를 어떻게 할거냐? 적절한 빈 공간에 찾아 넣을래? → 이건 비싼 operation. free area에 compact format으로 옮기는게 훨씬 효율적이다. 그래서 memory space를 block으로 나눔.

1. concurrent. 재배치 필요한 애들을 relocation set에 넣음
2. **STW**. relocation set의 모든 root reference를 재배치하고, 그들의 reference를 업데이트 해줌
3. concurrent. relocation set의 남은 애들 재배치. old/new 주소를 forwarding table에 저장.
4. 나머지 reference 재작성은 다음 marking 단계 또는 load barrier에서 발생함. object tree를 두 번 탐색할 필요가 없음.

### Remapping & Load barriers

relocation 단계에서 대부분의 재배치된 주소를 가르키는 reference를 업데이트 하지 않았었음. 여기서 그 문제를 풀어줌.

Load barrier : relocated된 오브젝트의 reference point를 고쳐주는 역할(remapping을 통해)

application이 reference를 로드할 때 load barrier를 트리거 하게 된다.

1. *remap* bit가 1인지 확인. 만약 1이면 reference가 최신으로 간주
2. referenced object가 relocation set에 있는지 확인. 만약 없다면 relocate 되지 않은 것. 따라서 remap bit를 1로 설정
3. 이제 우리가 접근하고자 하는 object는 relocation의 대상이었던 애라는 걸 알 수 있음. 유일한 문제는 relocation이 발생 했는지 아닌지이다. 
    1. 객체가 relocate 되었다면 다음 단계로
    2. 그렇지 않으면 relocate 실행하고, forwarding table에 entry 생성
4. 이제 오브젝트는 relocate 되었다는 것을 알게됨. ZGC, 또는 바로 이전(3단계), 또는 load barrier가 이전에 hit 함에 의해. 오브젝트의 새 주소로 reference를 업데이트 하고, remap bit를 설정하고, reference를 리턴한다.

![Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%2010.png](Garbage%20Collection%205f95d2d575cf4802b5737af62cdf2e36/Untitled%2010.png)

### Limitations of ZGC

class unloading 을 지원하지 않음. ClassUnloading 및 ClassUnloadingWithConcurrentMark 를 지원하지 않음.

JVMCI(i.e. Graal) 지원하지 않음. - JVM Compiler Interface

참조 : [https://www.baeldung.com/graal-java-jit-compiler](https://www.baeldung.com/graal-java-jit-compiler)

JIT Compiler : java → JVM bytecode. 근데 이걸 바로 실행 못함. 그래서 JVM은 이 bytecode를 interpret 해야 함. 그러나 interpreters는 일반적으로 native code보다 느림. 그래서 JIT(Just-in-time) compiler가 이를 machine code로 변경해서 더 빠르게 해줌

Graal - written in java, high-performance JIT compiler

JVMCI : JDK9 부터. 얘가 실제로 하는 역할은 standard tiered compilation을 제외하고, 우리의 새로운 compiler(Graal 같은) 애를 JVM쪽 변경 없이 추가해줄 수 있음.

# GC 선택

- Serial : 100MB 이하의 small data, single processor, no pause-time requirements
- Parallel : application performance 가 중요, no pause-time requirements 또는 수 초 pause 가 허용되는 경우
- G1 : Response time이 전체 throughput보다 중요한 경우, pause time을 짧게 유지해야 하는 경우.
- Z : response time이 가장 높은 우선순위, 매우 큰 heap size

7장

1. 트레이드오프
2. 동시 GC 이론
    1. JVM 세이프포인트
    2. 삼색 마킹
3. CMS
4. G1
5. 그 외 수집기

8장 : 로깅 관련은 제외

1. GC 로깅 개요
2. 로그 파싱
    1. 로그 샘플
    2. 툴
3. GC 기본 튜닝
4. Parallel, CMS, G1 쪽 옵션들
5. jHiccup

netty 방식

ZGC 에 대해서

- [https://www.baeldung.com/jvm-zgc-garbage-collector](https://www.baeldung.com/jvm-zgc-garbage-collector)
- [https://huisam.tistory.com/entry/jvmgc](https://huisam.tistory.com/entry/jvmgc)
- [https://wiki.openjdk.java.net/display/zgc/Main](https://wiki.openjdk.java.net/display/zgc/Main)

GC 선택

- [https://medium.com/leadkaro/z-garbage-collector-zgc-in-java-14-bd8a2fff4943](https://medium.com/leadkaro/z-garbage-collector-zgc-in-java-14-bd8a2fff4943)