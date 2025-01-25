## 1. Pipelining Analogy
Pipelining: 여러 명령어가 중첩되어 실행되는 구현 기술
Pipelining 방법론의 목적: 병렬처리로 성능 향상을 기대하는 것임
만약 **모든 단계(stage)가 거의 같은 시간이 걸리고, 할 일이 충분히 많다면, 파이프라이닝에 의한 **속도 향상은 파이프라이닝 단계 수(stage 수)와 같음**
### 할 일이 단계 수에 비해 충분하지 않을 경우우
* 아래의 그림에서 4배(stage 수)만큼이 아니라 2.3배 빠른 이유: 묶음이 4개밖에 없기 때문임 (즉, 할일이 파이프라인 단계 수에 비해 많지 않을 경우)
* 이렇게 할일이 충분하지 않을 경우에는, 시작 시간과 마무리 시간이 성능에 영향을 미침 
![pipelining_similar_cleaning](./pipelining_similar_cleaning.png)
## 2. MIPS Pipeline
### 5개의 Pipeline 단계
1. IF: 메모리로부터 Instruction Fetch (메모리로부터 명령어 로드)
2. ID: Instruction Decode & register read (명령어 해독 & 레지스터 읽기)
3. EX: EXcute operation / calculate address (연산 수행 / 주소 계산)
4. MEM: access MEMory operand (메모리 내부 피연산자에 접근)
5. WB: Write result Back to register (레지스터에 결과를 다시 기록)

## 3. Pipeline Performance
### Pipeline화된 data path에서의 실행 시간과 Single-cycle의 data path에서의 실행 시간 비교
![pipelined_vs_single-cycle_time_comparison](./pipelined_vs_single-cycle_time_comparison.png)

![single-cycle_vs_pipelined](./single-cycle_vs_pipelined.png)
## 4. Pipeline Speedup
* 만약 모든 단계들이 비슷한 처리시간을 갖는다면, '단계 수'배의 성능 향상 기대
* 모든 단계가 균등하지 않다면 속도 향상의 효과는 더 적을 것 => 가장 느린 단계에 맞춰짐
* 속도 향상은 Throughput을 늘려서 얻은 결과 (병렬적으로 Instruction들을 실행)
* 지연 시간(각 Instruction 하나를 실행하는데 소요되는 시간) 자체는 줄어들지 않음

## 5. Pipelining and ISA(Instruction Set Arch) Design
* MIPS ISA는 Pipelining에 적합한 구조
    * 모든 Instruction: 32비트라서 한 Cycle안에 Fetch하고 Decode하기 쉬움
* 적은 종류의 규격화된 Instruction Format
    * 한번에 Decode하고 레지스터를 Read할 수 있음
* Load/Store Addressing
    * 주소 계산은 3번째 단계에서, 메모리 접근은 4번째 단계에서 이루어짐 (위의 Pipeline 5단계 참고)
    * 즉, 단계별로 다른 작업을 수행
* Memory Operand의 정렬
    * Memory Access는 한 Cycle 안에 이루어짐

## 6. Hazards
* Hazard란?: 다음 Instruction이 다음 Clock Cycle에 실행될 수 없는 상황
    * 즉, 매 Cycle마다 Instruction을 실행해야 하는데, 그렇지 못하는 경우

### 3가지 종류의 Hazards
#### 1. Structure Hazard(구조적 Hazard)
* 정의: **Instruction을 수행하기 위해 필요한 HW 자원이 사용중인 경우**
* 예시: 단일 메모리를 사용하는 MIPS 파이프라인에서
    * Load/Store 명령은 데이터 접근을 위해 메모리에 접근
    * 그런데, Instruction Fetch도 Instruction을 가져오기 위해 메모리에 접근
    * 두가지가 동시에 진행 불가능, 한쪽은 대기하고 해당 Cycle을 지연시킴
* 해결 방안
    * 따라서, Pipeline화된 Datapath는 **Instruction Memory와 Data Memory가 분리**되어야 함
    * 혹은, **Instruction과 Data Cache를 분리**
* 대부분의 Structure Hazard는 자원 부족으로 인해 발생하며, 일반적으로 자원을 추가하면 해결됨

#### 2. Data Hazard(데이터 Hazard)
* 정의: **명령어를 실행하는 데 필요한 데이터가 아직 준비되지 않아** 계획된 명령어가 적절한 Clock Cycle에 실행될 수 없는 경우
* 예시: Instruction을 수행하기 위해 필요한 데이터가 있는데, **다른 Instruction이 데이터를 Read/Write하고 있는 경우**
* 해결 방안
    * **Forwarding**(전방 전달, Bypassing): 별도의 HW를 추가하여 정상적으로는 얻을 수 없는 값을 내부 자원으로부터 일찍 받아오는 것 (교재의 p318~319)
        * 그러나 예외적으로 적재-사용 Data Hazard(Load-Use Data Hazard)의 경우에는 Forwarding을 해도 한 단계 버블(지연)이 일어난다.
        * 교재 p320 문제 풀어보기

#### 3. Control Hazard(제어 Hazard)
* 정의: 어떤 Instruction 수행이 **다른 Instruction 결과에 의존하는 경우**, 제어 행동을 결정하는 것이 이전 Instruction에 의존하는 경우
* 해결 방안
    * 지연: 동작하는 것은 확실하지만 느림
    * 예측: Instruction 결과 예측이 틀린다면 다시 수행해야한다는 위험
        * 대부분의 컴퓨터가 분기 명령어를 다루기 위해 **예측**을 사용함
    * **지연 결정**: 분기 명령어에 의해 영향받지 않는 명령어를 지연 분기 명령어 바로 다음에 옮겨 놓으며, 분기 명령어는 이 안전한 명령어 이동을 고려하여 분기 주소를 변경
        * 분기 명령어가 실행되면, 프로세서는 분기된 **목적 주소(Target)**로 이동해야 하지만, 지연 슬롯에 넣은 명령어를 먼저 실행해야 함
        * 이를 위해 **분기 주소(Target)**를 원래 목표 주소보다 한 단계 뒤로 조정

## 7. Structure Hazards

## 8. Data Hazards

### Data Hazards --- Forwarding (aka Bypassing)
### Data Hazards --- Fowarding --- Load-Use Data Hazard
### Data Hazards --- Code Scheduling to Avoid Stalls

## 9. Control Hazards

### Control Hazards --- Stall on Branch
### Control Hazards --- Branch Prediction
### Control Hazards --- Branch Prediction --- MIPS with Predict Not Taken
### Control Hazards --- Branch Prediction --- 좀 더 현실적인 Branch Prediction

## 10. Pipeline Summary
* Pipelining은 **Instruction의 Throughput(명령어의 양양)을 늘림으로써 성능을 향상시키는 방법**
    * 한 명령어의 실행 속도가 아닌, 동시에 여러 명령을 가능하게 하는 양을 늘린 것
    * 병렬적으로 여러 명령을 실행
    * 각 Instruction은 같은 지연시간을 가짐
* Hazard에 주의해야 함
    * Structure Hazard (필요한 HW 자원이 사용중)
    * Data Hazard (필요한 데이터 준비 안됨)
    * Control Hazard (다른 Instruction 결과에 의존)
* Instruction Set의 설계 방식은 Pipeline 구현의 복잡도에 영향을 미침
    * ISA(Instruction Set Architecture) 복잡도 => Pipeline 구현 복잡도