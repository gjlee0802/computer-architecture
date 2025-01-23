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

## 5. Pipelining and ISA Design

## 6. Hazards

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