# 4.7. Data hazards: Forwarding vs Stalling
## 1. Data Hazards in ALU Instructions
다음 순서의 Instructions를 살펴보자  
　sub　**$2**, $1, $3  
　and　$12, **$2**, $5  
　or 　 $13, $6, **$2**  
　add　$14, **$2**, **$2**  
　sw 　 $15, 100(**$2**)  

* 우리는 Forwarding(bypassing)으로 Data Hazard를 해결할 수 있음
    * 그러면 Forwarding이 필요한 때는 어떻게 알 수 있는가(detect)?

## 2. Data Dependencies & Forwarding

### RECAP: Data Dependency (4.5. An Overview of Pipelining으로 돌아가서)
* 데이터 의존성(Data Dependency)는 RAW, WAW, WAR의 3가지 경우가 존재
    * 직전의 결과와 상관이 있을 때, Stall이 발생하는 Data Hazard는 RAW(Read After Write)에서 발생
    * RAW에서 발생하는 Stall:
        * 읽기(Read) 작업이 쓰기(Write) 작업보다 먼저 실행되면 데이터가 틀려질 수 있음
        * 이 때문에 파이프라인이 잠시 멈추는(정지: stall) 현상이 발생
    * WAW와 WAR:
        * WAW와 WAR의 경우, 멀티쓰레드에서 동기화 문제와 비슷함
        * 데이터를 동시에 여러 명령어가 처리하면, 쓰기 순서가 틀어져서 결과가 달라질 수 있음
        * 병렬 구조에서는 이런 상황을 방지하기 위해 쓰기 순서를 제대로 관리해야 함

### Forwarding은 RAW만 해결할 수 있음
* 우리가 관심있는 Dependency는 RAW에 의한 의존성
    * WAW, WAR의 데이터 의존성은 Forwarding으로는 해결하지 못함
    * 즉, **Forwarding은 RAW 데이터 의존성만을 해결 가능함**
    * WAW와 WAR은 병렬식으로 누가 먼저 써버리느냐에 따라 결과가 달라질 수 있음 (병렬성에 민감한 문제)

![dependencies_and_forwarding](dependencies_and_forwarding.png)

* 위 그림에서 $2 값을 위주로 살펴보자
    * 다음 cycle 기준으로는 EX/MEM 레지스터가 생성된 값을 가지고 있고(해당 명령어는 MEM단계로 들어갔으니),
    * 다다음 cycle 기준으로는 EX/MEM 레지스터에는 이미 다른 값으로 덮혔지만,
    * MEM/WB 레지스터에 해당 값을 가지고 있음(해당 명령어는 WB단계로 들어갔으니)

## 3. Detecting the Need to Forward (Forwarding이 필요한 순간 감지)
* Pipeline을 통해 레지스터 번호를 넘겨준다(pass).
    * e.g., **ID/EX.RegisterRs**: ID/EX pipeline register의 **Rs(SourceRegister)를 위한** 레지스터 번호
* EX stage에서의 ALU 피연산자 레지스터 번호는 다음과 같음(R-format의 ALU)
    * ID/EX.RegisterRs, ID/EX.RegisterRt (Rs와 Rt)
### Data Hazard가 발생하는 4가지 경우
~~~
1. Fwd from EX/MEM Pipeline Register (EX 기준 2 Cycle 전에서의 Rd)
    1a. EX/MEM.RegisterRd = ID/EX.RegisterRs
    1b. EX/MEM.RegisterRd = ID/EX.RegisterRt
2. Fwd from MEM/WB Pipeline Register (EX 기준 1 Cycle 전에서의 Rd)
    2a. MEM/WB.RegisterRd = ID/EX.RegisterRs
    2b. MEM/WB.RegisterRd = ID/EX.RegisterRt
~~~
* 1a, 1b는 다음 Cycle Instruction의 EX의 피연산자들(`ID/EX.Register의 Rs / Rt`)이 이전 Cycle Instruction의 MEM의 Rd(`EX/MEM.RegisterRd`)와 같을 경우를 말함
    * 즉, WB를 위한 대상 레지스터에 값이 쓰이고, 다른 명령어가 그 값을 읽어야하는데, 아직 값이 쓰이지 않아서
    * 1 Cycle 전의 Instruction에서, EX 직후에서 "계산된 Rd에 쓰일 값"을 끌어와 사용
* 2a, 2b는 다음 Cycle Instruction의 EX의 피연산자들(`ID/EX.Register의 Rs / Rt`)이 이전 Cycle Instruction의 WB의 Rd(`MEM/WB.RegisterRd`)와 같을 경우를 말함
    * 즉, WB를 위한 대상 레지스터에 값이 쓰이고, 다른 명령어가 그 값을 읽어야하는데, 아직 값이 쓰이지 않아서
    * 2 Cycle 전의 Instruction에서, MEM 직후에서 "Rd에 쓰일 값이 흐르던 것"을 끌어와 사용

### Forwarding이 이루어질 경우의 Control Signal과 Pipeline Register Rd
* Forwarding은 이전의 Instruction에서 레지스터에 값이 쓰여지지 않는 것들을 해결하는 것 (늦게 쓰여져서)
    * Pipeline Register에 함께 전해지는 Control Signal을 살펴보면, RegWrite 신호를 찾아볼 수 있음
        * EX/MEM.RegWrite, MEM/WB.RegWrite
* Write 대상이므로, Write 대상 레지스터 번호를 나타낼 Rd도 $zero가 아닐 것임 (0번 레지스터에는 Write 불가능)
    * EX/MEM.RegisterRd =/ 0
    * MEM/WB.RegisterRd =/ 0

## 4. Forwarding Paths
Forwarding을 위해 경로 및 장치를 추가하면 아래와 같음  
![forwarding_paths](./forwarding_paths.png)  
* Forwarding 감지를 위해 현재 Cycle의 ID/EX.Register의 Rs, Rt(`파란색 부분`)를 Forwarding Unit(`보라색 부분`)에 전달
    * 현재 **피연산자 Rs, Rt가 제대로 준비되었는지**가 Data Hazard와 Forwarding의 핵심이므로, 현재 Cycle은 피연산자가 필요한 실행단계인 **EX Stage**임
* **1 Cycle 전의 Instruction과 2 Cycle 전의 Instruction에서**(둘 다 아직 덜 끝남) **쓰여질 목표 레지스터(Rd)가 현재 Rs나 Rt와 같다면, 레지스터에는 값이 아직 제대로 쓰이지 않았을 것임을 의미**
* 이전 Cycle의 Rd를 보존하기 위해, Rd를 파이프라인 레지스터에 전달하도록 함(`빨간색 부분`)
    * 2 Cycle 전의 Rd는 EX/MEM.RegisterRd에
    * 1 Cycle 전의 Rd는 MEM/WB.RegisterRd에
* Forwarding Unit(`보라색 부분`)에서는,
    * 현재 Cycle의 Rs / Rt와 EX/MEM.RegisterRd 혹은 MEM/WB.RegisterRd가 같은 번호를 가지는지 확인함
        * 만약, EX/MEM.RegisterRd, MEM/WB.RegisterRd 모두 Rs, Rt와 일치하지 않으면 상관없음 (Forwarding 안함)
        * 만약, EX/MEM.RegisterRd가 Rs / Rt와 일치한다면, EX 직후인 ALU 결과값(`녹색 부분`)을 가져와서 사용함
        * 만약, MEM/WB.RegisterRd가 Rs / Rt와 일치한다면, WB로 Write하려는 값(`녹색 부분`)을 가져와서 사용함
    * Forwarding Unit(`보라색 부분`)은 Control Signal(`보라샌 선 부분`)을 3x1의 MUX(`주황색 부분`)들로 전달하여 ALU의 피연산자로, ID 단계의 Register 값을 사용할지(Forwarding 없는 일반적인 경우), Forwarding으로 가져온 값을 사용할지(Forwarding하는 경우)를 제어함

    ## 5. Datapath with Forwarding
    전체 Datapath는 아래와 같음  
    ![datapath_with_forwarding](./datapath_with_forwarding.png)  
    