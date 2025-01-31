# 4.8. Control Hazards
["4.5. An Overview of Pipelining" 복기하기](https://github.com/gjlee0802/computer-architecture/blob/main/4_5-an-overview-of-pipelining.md#9-control-hazards)

## 1. Branch Hazards
* branch의 결과가 MEM 단계에서 결정된다면?
* Control hazard 파트에서,
    * branch 결과를 알기 위해 기다리는 것 대신 "아닌 것으로 가정"하여 연속된 명령어를 수행하다가 아니면 그대로, 맞으면 제대로 명령어 가져와서 수행하기로 함
    * 만약 결과가 맞으면 그동안 미리 예측하여 수행해둔 명령어는 버려야 함
    * 제어 신호를 0으로 하여 비워야 함 (flush)

## 2. Reducing Branch Delay
예측에 실패하더라도 수행해둔 명령어가 버려지는 단계를 줄여보자  

* branch 결과 구하는 것을 ID 단계에서 할 수 있도록 하드웨어를 옮김 (ID Stage에 아래 하드웨어가 구성됨)
    * target address adder (대상 주소를 `PC+4+Offset(address*4) 해주던 ALU` 및 `Left shift`, `Sign extend`)
    * register comparator (ALU로 비교해서(빼서) zero인지 봤던)

* 예시: branch
36: sub $10, $4, $8  
40: **beq** $1,  $3,  **7**  
44: and $12, $2, $5  
48: or  $13, $2, $6  
52: add $14, $4, $2  
56: slt $15, $6, $7  
...  
72: lw  $4, 50($7)     #(40+4)+**7***4=72  
  
![control_hazards_reducing_branch_delay_datapath_1](./control_hazards_reducing_branch_delay_datapath_1.png)
![control_hazards_reducing_branch_delay_datapath_2](./control_hazards_reducing_branch_delay_datapath_2.png)
* beq는 다음 stage들도 넘어가도 딱히 할 일은 없음
* 예측이 맞다면 상관없이 다음 명령어 계속 수행
* 예측이 틀리고 분기로 이동(jump)해야한다면,
    * 분기 대상 주소를 PC에 넣어 해당하는 명령어를 IF(Fetch)할 수 있도록 하고,
    * 이전에 IF 단계에서 Fetch한 직후 명령어는 ID로 들어오지만 폐기함
* 기존 모델에서는 예측으로 실행해둔 여러 Instruction들을 버리면서 여러 Cycle의 Bubble이 발생하지만,
    * 하드웨어 구성을 통해 **ID 단계에서 branch 결과를 구하도록** 하여, **단 1번(IF 단계였던 직후 명령어)의 bubble 만으로 줄일 수 있음**

## 3. RECAP: Data Hazard의 정의는?
Data Hazard는 이전 명령어가 데이터를 읽거나 쓰는 것이 완료될 때까지 기다려야 하는 상황 (데이터 의존성에 의해)

## 4. Data Hazards for Branches
![data_hazards_for_branches_1](./data_hazards_for_branches_1.png)
* 만약 branch에서 비교하려는 레지스터가 2 Cycle 전이나 3 Cycle 전의 ALU Instruction 결과(Rd)라면 (아직 레지스터에 값이 쓰이지 않음),
    * RAW 데이터 의존성에 의한 것이므로 Forwarding을 통해 해결 가능
        * 이전에 본 forwarding은 현재 cycle이 EX stage였음(ALU에 필요한 피연산자)
        * 그런데 이번에 보는 branch의 비교 ALU는 ID 단계에 있기 때문에, (1 Cycle / 2 Cycle 전 단계가 아닌) 2 Cycle / 3 Cycle 전 단계에 대해 forwarding 가능함
        * (1 전 명령어는 이제 막 EX를 수행하는 중이고, 2 전 명령어는 EX/MEM, 3 전 명령어가 MEM/WB)  

![data_hazards_for_branches_2](./data_hazards_for_branches_2.png)
* 만약 비교하려는 레지스터가
    * 1 Cycle 전의 ALU Instruction의 결과(`ID/EX.RegisterRd`)이거나,
    * 2 Cycle 전의 load로 불러온 값을 쓸 레지스터(`EX/MEM.RegisterRt`)라면,
    * forwarding할 수 없고, stall이 발생할 수 밖에 없음

![data_hazards_for_branches_3](./data_hazards_for_branches_3.png)
* 1 Cycle 전(직전) 명령어가 산술(add)일 때, 이제 EX 들어가려하기 때문에 값이 없을 것임
    * branch의 피연산자인 ALU의 결과는 1번의 Stall이 필요
* 2 Cycle 전 명령어가 load일 때, 이제 MEM 들어가려하기 때문에 값이 없을 것임
    * branch의 피연산자인 MEM read는 2번의 Stall이 필요

## 5. Dynamic Branch Prediction
* 초대형(Stage가 아주 많은) Pipeline일수록, branch 패널티(stall)은 중요해짐(커짐)

* 동적 예측(Dynamic Prediction)을 사용하는 것이 바람직함

## 6. 1-Bit Predictor: Shortcoming(결점, 단점)
## 7. 2-Bit Predictor
## 8. Calculation the Branch Target(address)