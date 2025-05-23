# 면접 기출 예시
* 외부에서 수집한 자료, 본인이 임의로 만든 개념적인 질문을 포함함
* 대학원 면접을 대비하기 위한 자료임
* **서울대 컴퓨터공학부** 및 **포항공대 컴퓨터공학과**는 2가지의 과목을 선택하여 면접을 보도록 되어있으며, 그림과 함께 심층적으로 묻는 문제가 출제되는 것으로 알고 있음 [심층 문제로 이동](https://github.com/gjlee0802/computer-architecture/blob/main/interview_questions/4-the-processor.md#pipeline-hazards-%EC%8B%AC%EC%B8%B5-%EB%AC%B8%EC%A0%9C)
* **카이스트 전산학부**는 컴퓨터 구조 뿐만 아니라 모든 과목에서 전반적인 범위에 걸쳐 질문을 하며, 개념 질문들, 핵심 개념을 위주로 출제되는 것으로 알고 있음 [개념 질문으로 이동](https://github.com/gjlee0802/computer-architecture/blob/main/interview_questions/4-the-processor.md#%EA%B0%9C%EB%85%90-%EC%A7%88%EB%AC%B8)

## 개념 질문
### ISA (Instruction set architecture)
#### 1. CISC와 RISC의 차이점은 무엇인가?
~~~
CISC는 복잡하지만 코드 밀도가 높고, RISC는 단순하고 성능 최적화에 유리
"코드 밀도가 높다" == "적은 명령어와 작은 코드 크기로 동일한 작업을 수행할 수 있음"
~~~
| **구분**          | **CISC** (x86) | **RISC** (ARM, RISC-V) |
|------------------|--------------|----------------|
| **명령어 복잡성** | 복잡, 가변 길이 | 단순, 고정 길이 |
| **명령어 수**    | 많음 | 적음 |
| **메모리 접근**  | 명령어에서 직접 가능 | Load/Store 방식 |
| **실행 속도**    | 상대적으로 느림 | 빠름 (파이프라이닝 최적화) |
| **하드웨어 복잡도** | 높음 | 낮음 |
| **전력 소모**    | 높음 | 낮음 (모바일, 임베디드에 유리) |
| **대표 CPU**     | Intel, AMD | ARM, Apple M-series, RISC-V |

#### 2. Register File과 Cache의 차이는 무엇인가?
~~~
Register는 CPU 내부의 가장 빠른 저장 공간, Cache는 메모리와 CPU 사이의 임시 저장 공간임

Register는 프로세서가 직접 명령어를 수행할 때 사용하는 데이터를 저장함

Cache는 최근 사용한 데이터를 저장하여 메모리 접근 속도를 줄임 (CPU-메모리 병목 해소)
Cache 계층으로는 L1, L2, L3 캐시가 있음 (L1이 가장 빠르고, L3가 상대적으로 느림)
~~~

| **구분**         | **Register File** | **Cache** |
|-----------------|------------------|-----------|
| **위치**        | CPU 내부 | CPU와 메모리 사이 |
| **속도**        | 가장 빠름 | 빠름 (RAM보다 빠름) |
| **용량**        | 매우 작음 (수십 개) | 상대적으로 큼 (KB~MB) |
| **역할**        | 명령어 실행을 위한 데이터 저장 | 자주 사용하는 데이터 임시 저장 |
| **접근 속도**   | 나노초(ns) 수준 | 나노초~마이크로초(ns~μs) 수준 |
| **구성 방식**   | 일반 레지스터, 특수 레지스터 | L1, L2, L3 Cache 계층 |

#### 3. Intel과 AMD가 만드는 CPU의 공통점과 차이점은 무엇인가?
* ✅ 공통점
    * x86 아키텍처 기반 → CISC 구조 사용
    * 64 비트 프로세서 지원
    * 멀티코어 / 하이퍼스레딩 지원
        * 하이퍼스레딩은 CPU의 물리 코어를 논리적으로 2개의 코어처럼 활용하는 기술
* ✅ 차이점
    * Intel은 싱글 코어 성능과 자체 공정 강점, AMD는 멀티코어와 가성비 강점


### Pipelined Architecture
#### 1. 파이프라이닝은 왜 하는가?
~~~
Instruction들을 병렬적으로 처리하여 속도를 향상시키기 위함

병렬처리로 성능 향상을 기대하는 것임 만약 모든 단계(stage)가 거의 같은 시간이 걸리고, 할 일이 충분히 많다면, 
파이프라이닝에 의한 속도 향상은 파이프라이닝 단계 수(stage 수)와 같음
~~~
#### 2. 파이프라인 해저드(Pipeline Hazard)란 무엇인가? 그리고 종류에는 무엇이 있는가?
~~~
다음 Instruction이 다음 Clock Cycle에 실행될 수 없는 상황
즉, 매 Cycle마다 Instruction을 실행해야 하는데, 그렇지 못하는 경우
~~~
- Structure Hazards (Structural)
    - **정의**: 서로 다른 명령어들이 하드웨어 리소스를 동시에 사용하려고 할 때 구조적으로 생기는 문제
    - **예시**: 예를 들어 Load/Store 명령어가 메모리에 접근하는 동시에 Instruction Fetch를 하려 하는 다른 Instruction가 있어, 동시적으로 하드웨어에 접근하여 발생하는 지연 문제가 있음
    - **해결**: Instruction Memory와 Data Memory를 구조적으로 분리시켜야 하거나, Instruction과 Data Cache를 분리해야 함
- Data Hazards
    - **정의**: 사용하려는 데이터가 준비되지 않아 데이터 의존성으로 인해 지연이 발생하는 문제
    - **예시**: 바로 직전 Instruction의 연산 결과가 레지스터에 쓰이지 않는 Read After Write 문제 같은 경우가 발생할 수 있음
    - **해결**: 위의 예시 같은 경우는 **Forwarding**으로 해결하며, Forwarding으로 해결되지 않는 예시로는 메모리에서 데이터를 읽어야 값을 가져올 수 있어서 값이 존재하지 않는 예가 있는데, 이러한 경우 **Code Scheduling**을 통한 Stall 방지를 고려할 수 있음
- Control Hazards
    - **정의**: 분기로 인해 Instruction flow가 바뀜으로 인해 지연이 발생하는 문제
    - **예시**: 예를 들어 beq 같은 조건부 분기(branch)가 있다고 할 때, 바로 다음 Instruction을 처리하기 위해 수행 중일 수 있는데, branch로 인해 다른 Instruction 수행을 해야할 수 있음
    - **해결**: Branch Prediction을 도입하여 간단한 방법으로는 항상 틀린 것(아닌 것으로)으로 예측(가정)할 수 있으며(다음 Instruction이 수행될 것으로 예측), 다른 현실적인 방법으로는 Hisory 테이블을 활용하여 이전의 branch 결과를 참고하여 예측하는 방법이 있음

#### 3. 파이프라이닝에서 Branch Prediction의 역할은 무엇인가?
~~~
Branch Prediction은 Control Hazards를 위한 분기 결과를 예측함으로써 지연을 줄이려는 방법으로, 
단순한 방법으로는 항상 분기 결과가 틀린 것(아닌 것)으로 가정하는 것이 있으며, 
현실적인 방법으로는 history table을 이용하여 이전의 분기들을 참고해서 예측하는 Dynamic Branch Prediction이 있음
~~~

#### 4. Branch Prediction에서 단순히 아닌 것으로 가정하는 방법을 사용했을 때의 문제점은? (굳이 Dynamic Branch Prediction을 사용하는 이유)
~~~
초대형(Stage가 아주 많은) Pipeline일수록, branch 패널티(stall)은 중요해짐(커짐)
이런 경우를 대비하여 Dynamic Branch Prediction을 활용하는 것이 적절함
~~~
### Exceptions
#### 1. Exception의 정의는?
~~~
CPU 내부에서 발생한, 예기치 못한 상황
~~~
#### 2. Exceptions 과 Interrupts의 차이는 무엇인가?
~~~
Exception(예외)은 CPU 내부(프로그램 내부)에서 발생하지만,
Interrupt(인터럽트)은 외부 장치나 OS로부터 발생함
~~~
* Exception(예외)은 **CPU 내부(프로그램 내부)에서** 발생
    * e.g., undefined opcode, overflow, syscall, ...
* Interrupt(인터럽트)은 **외부 I/O Controller(CPU 외부의 장치, OS 등...)로부터** 발생

#### 3. Exception 처리 과정을 설명하시오.
~~~
예측 실패한 branch의 처리와 유사함
1. Exception이 발생한 Instruction과 이후에 수행 중이던 Instruction들은 Flush하고,
2. EPC와 Cause Register의 값을 세팅하여 Exception이 발생한 주소, 발생 원인을 저장하고,
3. handler 명령어로 jump하여 handler가 처리하도록 함
~~~

## 💪 Pipeline Hazards 심층 문제
### 1. 5단계의 Pipeline에서 아래의 코드가 수행된다고 하자. (SSU 19년도 기출)
~~~
add $3, $1, $2      ...(1)
and $12, $5, $3     ...(2)
or  $13, $3, $6     ...(3)
sub $14, $3, $$2    ...(4)
sw  $15, 100($3)    ...(5)
~~~
#### 1.1. 위 코드 수행과정에서 발생하는 hazard를 설명하시오.
~~~
🎯 1번 명령어와 2번 명령어 사이에 $3으로 인한 Data hazard가 발생한다. 
(2)를 수행함에 있어서 EX 단계(ALU)에서 $3에 해당하는 값이 필요한데, 
add 명령어의 결과로 얻어지는 데이터이기 때문에 
RAW(Read After Write) 데이터 의존성으로 인해 hazard가 발생한다.
~~~
#### 1.2. 위에서 발생한 hazard를 Forwarding 방법만으로 해결하려고 한다. 가능한지 여부를 답하시오. (1) Yes라면, Forwarding 과정을 구체적으로 설명하시오. (2) No라면, 그 이유를 설명하고 추가적으로 해결하는 방법을 설명하시오.
~~~
🎯 Yes, Forwarding 방법만으로 해결할 수 있다.
~~~
* (2)번 명령어의 경우, 
    * `$3`이 **EX 단계에서 필요**하므로, (1)번 명령어가 MEM 단계에 있을 때 **EX/MEM 레지스터의 값을 포워딩**하여 해결함
* (3)번 명령어의 경우,
    * (1)번 명령어가 WB 단계에 있을 때 **MEM/WB 레지스터로부터 데이터를 포워딩**할 수 있음


#### 1.3. 위에서 발생한 hazard를 stall 방법(bubble 추가)만으로 해결하려 한다. 위 코드를 모두 수행하기 위해 bubble이 총 몇 개 필요한지 설명하시오.

~~~
🎯 2개 필요하다. 
(2)번 명령어가 ID 단계에서 $3의 값을 읽으려면,
(1)번 명령어의 WB 단계에서 쓰기가 끝난 후 읽어야하는데, 
2번의 Stall로 미뤄져야 가능하기 때문이다.
~~~

#### 1.4. 아래와 같이 (2)의 명령어가 `lw $2, 4($4)`로 바뀌었다고 하자. 이로 인해 새로 생긴 hazard를 설명하시오.
~~~
add $3, $1, $2      ...(1)
lw  $2, 4($4)       ...(2)
or  $13, $3, $6     ...(3)
sub $14, $3, $2     ...(4)
sw  $15, 100($3)    ...(5)
~~~
~~~
🎯 (2)와 (4) 사이에 Load-use hazard가 생긴다. 
(4)번 명령어는 $2의 값을 EX 단계에 피연산자로 사용해야 하는데, 
(4)번 명령어가 EX 단계에 들어섰을 때, 
(2)번 명령어는 WB 단계에 들어섰을 것이다.
따라서 (2)번 명령어는 WB 단계를 마치지 않아,
$2의 값을 얻을 수 없다.
~~~

#### 1.5. 새로 생긴 hazard를 Forwarding 방법 만으로 해결하려 한다. 가능한지 여부를 답하시오. (1) Yes라면, Forwarding 과정을 구체적으로 설명하시오. (2) No라면, 그 이유를 설명하고 추가로 해결하는 방법을 설명하시오
~~~
🎯 No, Forwarding만으로는 해결할 수 없다.

이 문제는 Load-use hazard로, lw 명령어의 결과가 MEM 단계 이후에야 유효하게 되므로,

그 다음 명령어인 sub에서 EX 단계에서 이 값을 사용하려 해도 아직 준비되지 않아 Forwarding이 불가능하다.
~~~

이 경우 해결책은 다음과 같다:
~~~
1. stall을 삽입하여 sub 명령어가 EX 단계에 진입하기 전에 lw 명령어가 WB 단계까지 도달하도록 한다.

2. 또는 명령어 재배치, Code Scheduling으로 해결할 수 있다.
~~~

#### 1.6. 위에서 추가로 발생한 hazard를 stall 방법만으로 해결하려고 한다. bubble이 총 몇 개 필요한지 설명하시오.
~~~
🎯 bubble은 2개 필요하다. 
2개의 Bubble이면 (4)번 명령에서 $2의 값이 필요한 시점(ID 단계)에,
(2)번 명령의 WB 단계를 마칠 수 있기 때문이다.

(2) IF ID EX MEM WB
(3)    IF ID EX  MEM WB
(-)       -  -   -   -  -
(-)          -   -   -  -  -
(4)              IF  ID EX MEM WB
~~~

-----

### 2. '그림 1'은 5단계 pipeline에서 data hazard를 해결하기 위해 forwarding unit과 hazard detection unit이 추가된 회로도이다. (SSU 17년도 기출)
![datapath_with_hazard_detection](../image_files/datapath_with_hazard_detection.png)  
그림 1. data hazard를 해결하기 위해 forwarding unit과 hazard detection unit이 추가된 회로도
#### 2.1. 아래 코드를 그림 1 회로에서 수행할 경우, 발생하는 모든 hazard에 대해 구체적으로 설명하시오.
~~~
add $5, $2, $1      ...(1)
lw  $3, 4($5)       ...(2)
lw  $2, 0($2)       ...(3)
or  $3, $5, $3      ...(4)
sw  $3, 0($5)       ...(5)
~~~
~~~
🎯 Hazard 1) 첫번째 Hazard로,
1번 명령어와 2번 명령어 사이에 $5 때문에 RAW Data Dependency로 인한 Data Hazard가 발생한다.
$5의 값이 2번 명령어의 EX stage에 피연산자로 필요한데,
1번 명령어의 $5의 값은 연산을 거쳐 EX stage를 마쳤을 때의 결과 값이므로 EX/MEM 레지스터로부터 Forwarding해서 가져와야 한다.
2번 명령어가 EX stage에 진입할 때, 1번 명령어는 MEM stage에 진입하므로 Forwarding이 가능하다.

🎯 Hazard 2) 두번째 Hazard로,
2번 명령어와 4번 명령어 사이에 $3 때문에 Load-Use Data Hazard가 발생한다.
$3의 값이 4번 명령어의 EX stage에 피연산자로 필요한데,
2번 명령어의 $3의 값은 메모리에서 읽은(load) 값이므로 MEM/WB 레지스터로부터 Forwarding해서 가져와야 한다. 
2번 명령어가 WB stage에 진입할 때, 4번 명령어는 EX stage에 진입하므로 Forwarding이 가능하다.

🎯 Hazard 3) 세번째 Hazard로,
4번 명령어와 5번 명령어 사이에 $3 때문에 RAW Data Dependency로 인한 Data Hazard가 발생한다.
RAW Hazard는 "이전 명령어에서 값을 계산하여 갱신하는데, 다음 명령어에서 이를 사용해야 할 때" 발생하는 것이다. 
즉, 4번 명령어에서 $3을 새롭게 할당하는데, 5번 명령어에서 $3을 저장하려고 하므로,
5번 명령어가 4번 명령어의 결과를 정확하게 반영하려면 forwarding (data forwarding)이나 stall이 필요하다.
~~~

#### 2.2. '그림 1'에서 forwarding unit은 있지만, 만일 hazard detection unit이 없다면 위 코드를 문제없이 실행할 수 있는지 여부를 Yes/No로 대답하고 그 이유를 구체적으로 설명하시오.
~~~
🎯 Yes, Forwarding Unit만으로 정상적으로 실행 가능하며, Hazard Detection Unit이 없어도 문제되지 않는다.

✅ Forwarding Unit은 EX 단계에서 발생한 연산 결과를 바로 다음 명령어에 전달하여 RAW (Read After Write) Hazard를 방지하는 역할을 한다.
✅ 하지만 Load-Use Hazard의 경우 Forwarding만으로는 해결되지 않고 Stall이 필요할 수 있다.
✅ 이 코드에서는 3번 명령어(lw $2, 0($2))가 자연스럽게 Stalling 역할을 수행하여, 추가적인 Stall 없이 Forwarding만으로 해결 가능하다.
~~~
| 조건 | Forwarding Unit만으로 실행 가능 여부 | 이유 |
|------|------------------------|------|
| **3번 명령어가 없는 경우** (2번 → 4번 바로 실행) | ❌ **Stalling이 필요함** | Forwarding만으로 해결되지 않으므로 Hazard Detection Unit이 필요 |
| **3번 명령어가 있는 경우** (2번 → 3번 → 4번 실행) | ✅ **문제없이 실행 가능** | 3번 명령어가 자연스럽게 Stalling 역할을 수행하여 Forwarding만으로 해결 가능 |

#### 2.3. '그림 1'에서 forwarding unit은 없고 hazard detection unit만 가지고 위 코드를 수행하려고 한다. hazard detection unit에 어떤 입력과 출력 신호가 새로 필요한지를 설명하시오. 왜 이런 신호들이 필요한지 예를 들어 설명하시오.
~~~
✅ Forwarding Unit에서는 EX stage에서 Hazard를 감지 했는데, Hazard Detection Unit에서 Hazard를 검출하기 위해서 ID stage에서 비교해야 하는 것이 핵심이다!!

✅ Forwarding Unit은 EX stage에서 ID/EX.registerRs, ID/EX.registerRt 피연산자 정보, 직전(1 Cycle 전) 명령어의 목적지 레지스터 EX/MEM.registerRd, 2 Cycle 전 명령어의 MEM/WB.registerRd 목적지 레지스터를 비교하여 Hazard 검출될 때 "Forwarding"을 하는 역할이다.

🎯 유사하게, Hazard Detection Unit의 경우에는 기존의 EX stage에서 검출하기 위해 사용한 정보 대신에, ID stage에서 IF/ID.registerRs, IF/ID.registerRt, ID/EX.registerRd, EX/MEM.registerRd 를 비교하여 Hazard를 검출한다.
그러나, Hazard Detection Unit만으로 Forwarding은 수행하지 않으며, Stall만 추가할 수 있다.
~~~

#### 2.4. 위 '2.3.' 문제의 회로를 갖고 위 코드를 전부 수행하려고 한다. 첫 사이클부터 종료될 때까지 사이클별로 Stalling을 포함하여 명령어 수행 순서를 정리하여 설명하시오. 
~~~
✅ Hazard Detection Unit만 있는데, Stalling만 가능하다.
각각의 Hazard에 대해 몇번의 bubble이 필요한지 생각해보면 된다.
위의 Hazard1을 해결하려면 2개의 bubble이 필요하다.
위의 Hazard2를 해결하려면 1개의 bubble이 필요하다.
위의 Hazard3를 해결하려면 2개의 bubble이 필요하다.

🎯 이를 감안하여 명령어 수행 순서를 정리하자면,
1번 명령어 수행
bubble (Hazard : 1 - 2)
bubble (Hazard : 1 - 2)
2번 명령어 수행
3번 명령어 수행
bubble (Hazard : 2 - 4)
4번 명령어 수행
bubble (Hazard : 4 - 5)
bubble (Hazard : 4 - 5)
5번 명령어 수행
~~~

-----

### 3. 다음은 Control Hazard에 대한 질문이다. '그림 1'에서 아래 코드가 수행된다고 하자. 맨 앞의 숫자는 그 명령어가 저장된 주소이다. (SSU 17년도 기출)
![datapath_with_hazard_detection](../image_files/datapath_with_hazard_detection.png)  
그림 1. data hazard를 해결하기 위해 forwarding unit과 hazard detection unit이 추가된 회로도

~~~
30: beq $1,  $3,  5       ...(1)
34: or  $8,  $5,  $2      ...(2)
38: add $9,  $3,  $4      ...(3)
42: and $6,  $2,  $7      ...(4)
46: add $14, $2,  $2      ...(5)
...
54: sw  $2, 4($3)         ...(6)
~~~

#### 3.1. 30번지 beq 명령어에서 조건이 만족되어 branch 해야하는 상황을 그 명령어의 어느 수행단계에서 알게 되는가? 답을 쓰고 이유를 구체적으로 설명하시오. branch가 발생하는 경우 flushing해야 하는 명령어가 몇 개인지 위의 코드에서 구체적으로 밝히시오. Timing chart와 같은 그림을 그려서 발생하는 상황을 구체적으로 설명하시오.

~~~
✅ beq 조건부 명령어에서 $1과 $3이 같다는 것을 알게 되는 것은 EX stage이지만,
✅ 분기할 최종 주소를 계산하여 알게되는 것은 MEM stage이다.
✅ 이 MEM stage 다음 clock cycle에서 분기할 주소의 명령어를 가져온다.
분기할 최종 주소는 (30 + 4) + (5 * 4), 54번지인 6번 명령어이다.
🎯 아래의 Timing chart를 참고하면 3개의 명령어가 Flush되어 버려진다.
~~~
🎯 **Timing chart**:  
| PC        | CC1 | CC2 | CC3 | CC4 | CC5            | CC6 | CC7 | CC8 | CC9 |
|-----------|-----|-----|-----|-----|----------------|-----|-----|-----|-----|
| 30 (beq)  | IF  | ID  | EX  | MEM | WB             |     |     |     |     |
| 34 (or)   |     | IF  | ID  | EX  | ❌**Flushed** | -   |     |     |     |
| 38 (add)  |     |     | IF  | ID  | ❌**Flushed** | -   | -   |     |     |
| 42 (and)  |     |     |     | IF  | ❌**Flushed** | -   | -   | -   |     |
| **58 (sw)** |   |     |     |     | IF             | ID  | EX  | MEM | WB  |

**beq 명령어의 실행 과정을 정리하자면**,  
`beq $1, $3, offset` 명령어는 `$1`과 `$3`의 값이 같은 경우, `offset × 4` 바이트 만큼 이동하여 분기하는 명령어임
* EX (Execute) 단계에서:
    * 두 레지스터 값($1, $3)을 비교하여 조건이 만족하는지 결정함.
    * 즉, 분기 여부(branch taken or not taken)는 EX 단계에서 결정됨.
* MEM (Memory) 단계에서:
    * PC 값을 업데이트하고, 실제로 분기할 최종 주소를 결정.
    * 분기가 확정되면 파이프라인을 Flush하고, 새로운 명령어를 가져오기 시작.

#### 3.2. 아래 '그림 2'는 '그림 1'과 비교할 때 회로가 추가되었다. 어떤 부분에 어떤 기능이 추가됐는지와 이 회로가 추가된 목적을 설명하시오. (위에 쓰인 명령어는 무시)
![control_hazards_reducing_branch_delay_datapath_1](../image_files/control_hazards_reducing_branch_delay_datapath_1.png)
그림 2. 변경된 회로도

~~~
✅ branch할 주소를 계산하는 요소(adder)를 ID stage에서 가능하도록 옮겼다.
✅ 주소가 같은지 비교하는 요소(comparator)를 ID stage에서 가능하도록 옮겼다.
✅ IF.flush Control Signal이 추가되었다.
✅ 즉, 변경하기 전인 '그림 1'에서는 '3.1.' 문제에서 봤듯이 분기 여부는 EX stage에서 수행되고, 분기할 최종 주소를 결정하는 것은 MEM Stage에서 수행된다.

🎯 '그림 2'에서는 주소 계산을 ID stage에서 가능하게 하여 branch할 주소를 미리 ID stage에서 결정하게 됨으로써 flush 개수를 줄일 수 있다. ID stage에서 branch할 주소가 결정되기 때문에 ID stage 다음 Clock Cycle에서 branch할 주소의 명령어가 수행된다.
🎯 즉, 이 회로가 추가된 목적은 flush 개수를 줄여 지연(delay)를 줄이기 위함이다.
~~~

#### 3.3. '그림 2' 회로에서 위의 코드를 수행한다고 할 때 수행과정을 Timing chart와 같은 표를 그려서 발생하는 상황을 구체적으로 설명하시오. branch가 발생하는 경우 Flushing 해야 하는 명령어가 몇 개인지 위의 코드에서 구체적으로 밝히시오.
🎯 아래의 Timing chart를 참고하면 1개의 명령어가 Flush되어 버려진다.  
🎯 **Timing chart**:  
| PC        | CC1 | CC2 | CC3           | CC4 | CC5 | CC6 | CC7 |
|-----------|-----|-----|---------------|-----|-----|-----|-----|
| 30 (beq)  | IF  | ID  | EX            | MEM | WB  |     |     |
| 34 (or)   |     | IF  | ❌**Flushed** | -   | -   | -   |     |
| 58 (sw)   |     |     | IF            | ID  | EX  | MEM  | WB |

-----

### 4. 다음 코드는 5단계 파이프라인 구조의 RISC-V CPU에서 실행된다. 분기 예측 방식은 정적(Static) 방식이며, 항상 "분기 안 함(Not Taken)"으로 예측한다고 가정한다. 즉, 분기 명령이 나오면 항상 다음 PC = PC + 4로 예측한다. 분기 결과는 EXE 단계(ALU에서)에 가서야 확인할 수 있다. (POSTECH 22-23 First)
~~~
    LW x9, 0x0(x8)
    BEQ x9, x10, END    //It turns out the values of x9 and x10 differ
    ADD x9, x9, x9
END:ADD x10, x10, x10
~~~

✅ Not Taken으로 예측한다고 가정한 상태에서, x9과 x10이 다르기 때문에 Branch Prediction에 성공하여 Flush를 하지 않음  
✅ 만약, Branch Prediction에 실패한다면, BEQ의 EX에서 분기 여부가 판별되는 시점까지 인출된 명령어들이 Flush  

#### 4.1. data forwarding이 전혀 구현되어 있지 않은 경우(즉, 레지스터 파일 내부 포워딩도 없음) 이 코드를 실행하는 데 몇 사이클이 소요되는가?
~~~
5 stage는 순서대로, IF -> ID -> EX -> MEM -> WB

LW에서 값을 가져와야 하는 Load-Use Data Hazard 발생임

LW에서 x9의 값은 WB 단계를 거쳐야 얻어짐 (WB를 통해 메모리에 접근하여 읽은 값을 x9에 저장)
그런데 BEQ는 ID 단계에서 x9의 값을 얻어야 함
LW의 WB 단계 다음에 BEQ의 ID 단계가 수행되어야 하므로,
아래와 같이 3번의 Bubble이 필요함
 IF ID EX MEM WB (LW)
    -  -  -   -  - (Bubble)
       -  -   -  -  - (Bubble)
          -   -  -  -   - (Bubble)
              IF ID EX MEM WB (BEQ)
                 IF ID EX  MEM WB (ADD)
                    IF ID  EX  MEM WB (ADD)

🎯 따라서, 11 cycles 소요
~~~

#### 4.2. 모든 가능한 데이터 포워딩이 구현된 경우, 이 코드를 실행하는 데 몇 사이클이 소요되는가?
~~~
Load-Use Data Hazard임

BEQ의 EX 단계에서 비교가 이루어지므로, 
forwarding을 도입할 경우에는 비교 연산 때 x9 값을 가져오면 됨!
즉, ID에서는 x9 값으로 outdated 데이터를 읽어도 EX에서 forwarding을 통해 최신 값을 가져오는 것

Load-Use data hazard이므로, x9는 메모리에 접근해야만 얻을 수 있기에
MEM 이후에 가져올 수 있음
따라서 forwarding을 하더라도 한번의 stall이 발생

 IF ID EX MEM WB (LW)
    -  -  -   -   -   - (Bubble)
       IF ID  EX  MEM WB (BEQ)
          IF  ID  EX  MEM WB (ADD)
              IF  ID  EX  MEM WB (ADD)

🎯 띠라서, 9 cycles 소요 (마지막 ADD의 WB 단계까지)
~~~


## 💪 Branch Prediction 심층 문제

### 1. 다음 C 프로그램은 중첩 루프로 구성되어 있다. Outer Loop는 10번 반복되고, Inner Loop는 20번 반복된다:   (POSTECH 24-25 Second)
~~~
for (i = 0; i < 10; i++) {
    for (j = 0; j < 20; j++) {
        // do something;
    }
}
~~~
#### 1.1. 각 루프의 끝에 분기 명령어가 하나씩 존재하고, 그 외에는 코드에 다른 분기 명령어가 없다고 가정할 때, 다음 분기 예측기(branch predictor)들의 정확도를 비교하고, 주어진 프로그램의 실행 동작과 관련지어 그 이유를 설명하시오.
~~~
(1) Static predictor (always predicts “taken”)
(2) One-bit predictor
(3) Two-bit predictor using a saturating counter
~~~

✅ Taken과 Not Taken의 개념:
| 용어        | 뜻                                | 실행 흐름       |
|-------------|-----------------------------------|------------------|
| **Taken**   | 분기가 실행되어 **점프함**         | 루프 반복 계속    |
| **Not Taken** | 분기가 실행되지 않고 **직진함**     | 루프 종료, if문 통과 등 |

✅ Inner Loop와 Outer Loop의 Taken/Not Taken 횟수 분석:
* Inner Loop:
    * 한번의 Outer Loop당 20번 반복되어, 20개의 분기 발생
    * 실제 분기 결과: 19번 Taken + 1번 Not Taken
    * Outer Loop가 10번 반복되므로, 190번 Taken + 10번 Not Taken
* Outer Loop:
    * 9번 Taken + 1번 Not Taken

1️⃣ **Static Predictor** - 항상 Taken으로 예측
* Taken일 때는 맞고, Not Taken일 때는 틀림
* Inner Loop의 정확도: `190/200` = 95%
* Outer Loop의 정확도: `9/10` = 90%
* 🎯 전체 평균 정확도: `199/210` = 94.7%

2️⃣ **One-bit Predictor** - 최근 분기 결과 하나만 기억
* Loop의 마지막 Not Taken이 발생한 후, 다음 Loop의 첫 분기를 잘못 예측
* Outer Loop에 의해 다음 Inner Loop가 10번 반복될 때마다, 2번 연속으로 틀림
    ~~~
    i=0, j=0    Taken
    i=0, j=1    Taken
    ...
    i=0, j=19   Taken
    i=0, j=20   Not Taken(루프탈출) -> 틀림
    i=1, j=0    Taken -> 틀림
    ~~~
* Inner Loop는 10번 반복되므로, 10 x 2 = 20 번 틀림
* Inner Loop의 정확도: `180/200` = 90%
* Outer Loop의 정확도: `8/10` = 80%
* 🎯 전체 평균 정확도: `188/210` = 89.6%

3️⃣ **Two-bit Predictor** - 예측 상태를 4단계(Strong/Weak Taken, Strong/Weak Not Taken)로 기억
* 한 번 틀린다고 예측이 바뀌지 않음
* Static Predictor처럼 Loop 탈출하는 Not Taken에서 1번씩 틀림
* Inner Loop의 정확도: `190/200` = 95%
* Outer Loop의 정확도: `9/10` = 90%
* 🎯 전체 평균 정확도: `199/210` = 94.7%

-----

### 2. 다음과 같은 branch 연산의 결과가 주어졌다면, 기본적인 two-bit saturating counter를 이용한 branch predictor를 사용할 때 예측 결과의 정확도를 구하시오. 이때, counter는 2’b10으로 초기화되었고 2’b00은 strongly not-taken, 2’b11은 strongly taken을 의미한다. (POSTECH 24-25 First)
~~~
Branch 연산 결과: T, T, T, N, N, N, T, T, T, N, N
~~~

✅ 2-bit Saturating Counter Predictor의 상태표:  
| 상태 값 | 상태 이름         | 예측       | 실제 결과: Taken → 다음 상태 | 실제 결과: Not Taken → 다음 상태 |
|----------|------------------|------------|------------------------------|----------------------------------|
| 11       | Strong Taken     | Taken      | 유지 (11)                    | → Weak Taken (10)               |
| 10       | Weak Taken       | Taken      | → Strong Taken (11)          | → Weak Not Taken (01)           |
| 01       | Weak Not Taken   | Not Taken  | → Weak Taken (10)            | → Strong Not Taken (00)         |
| 00       | Strong Not Taken | Not Taken  | → Weak Not Taken (01)        | 유지 (00)                       |


처음에 초기화되었을 때: `10` (Weak Taken)  

| 순서 | 실제 결과 | 현재 상태        | 다음 상태           | 예측 성공 여부        |
|------|------------|------------------|----------------------|-----------------|
| 1    | T          | Weak Taken (10)  | Strong Taken (11)   | ✅ 성공          |
| 2    | T          | Strong Taken (11)| Strong Taken (11)   | ✅ 성공          |
| 3    | T          | Strong Taken (11)| Strong Taken (11)   | ✅ 성공          |
| 4    | N          | Strong Taken (11)| Weak Taken (10)     | ❌ 실패          |
| 5    | N          | Weak Taken (10)  | Weak Not Taken (01) | ❌ 실패          |
| 6    | N          | Weak Not Taken(01)| Strong Not Taken (00)| ✅ 성공        |
| 7    | T          | Strong Not Taken(00)| Weak Not Taken (01)| ❌ 실패        |
| 8    | T          | Weak Not Taken(01)| Weak Taken (10)     | ❌ 실패         |
| 9    | T          | Weak Taken (10)  | Strong Taken (11)   | ✅ 성공          |
|10    | N          | Strong Taken (11)| Weak Taken (10)     | ❌ 실패          |
|11    | N          | Weak Taken (10)  | Weak Not Taken (01) | ❌ 실패          |

🎯 따라서, 예측 결과의 정확도는 `5/11` = 약 45.4%  