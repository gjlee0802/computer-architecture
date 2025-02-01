# 면접 기출 예시
* 외부에서 수집한 자료, 본인이 임의로 만든 개념적인 질문을 포함함
* 대학원 면접을 대비하기 위한 자료임
* 서울대 컴퓨터공학부는 2가지의 과목을 선택하여 면접을 보도록 되어있으며, 그림과 함께 심층적으로 묻는 문제가 출제되는 것으로 알고 있음
* 카이스트 전산학부는 컴퓨터 구조 뿐만 아니라 모든 과목에서 전반적인 범위에 걸쳐 질문을 하며, 개념적인 정의 질문들, 핵심 개념을 위주로 출제되는 것으로 알고 있음

## 개념 질문
### ISA (Instruction set architecture)
* CISC와 RISC의 차이점은 무엇인가?
* Register File과 Cache의 차이는 무엇인가?
* Intel과 AMD가 만드는 CPU의 공통점과 차이점은 무엇인가?
### Pipelined Architecture
1. 파이프라이닝은 왜 하는가?
~~~
Instruction들을 병렬적으로 처리하여 속도를 향상시키기 위함
~~~
2. 파이프라인 해저드(Pipeline Hazard)란 무엇인가? 그리고 종류에는 무엇이 있는가?
~~~
파이프라인 해저드는 병렬적으로 처리함에 있어서 성능이 저하되는 사건을 말하며
아래의 세가지 종류가 있음
~~~
- Structure Hazards
    - 정의: 하드웨어 리소스를 사용함에 있어서 구조적으로 지연이 발생하는 문제
    - 예시: 예를 들어 동시적으로 하드웨어에 접근하여 발생하는 지연 문제가 있음
    - 해결: Instruction Memory와 Data Memory를 구조적으로 분리시켜야 함
- Data Hazards
    - 정의: 사용하려는 데이터가 준비되지 않아 데이터 의존성으로 인해 지연이 발생하는 문제
    - 예시: 바로 직전 Instruction의 연산 결과가 레지스터에 쓰이지 않는 Read After Write 문제 같은 경우가 발생할 수 있음
    - 해결: 위의 예시 같은 경우는 Forwarding으로 해결하며, Forwarding으로 해결되지 않는 예시로는 ALU 연산 결과를 바로 가져오지 못하고 메모리에서 데이터를 읽어야 값을 가져올 수 있는 예가 있는데, 이러한 경우 Code Scheduling을 통한 Stall 방지를 고려할 수 있음
- Control Hazards
    - 정의: 분기로 인해 Instruction flow가 바뀜으로 인해 지연이 발생하는 문제
    - 예시: 예를 들어 beq 같은 조건부 분기(branch)가 있다고 할 때, 바로 다음 Instruction을 처리하기 위해 수행 중일 수 있는데, branch로 인해 다른 Instruction 수행을 해야할 수 있음
    - 해결: Branch Prediction을 도입하여 간단한 방법으로는 항상 틀린 것(아닌 것으로)으로 예측(가정)할 수 있으며(다음 Instruction이 수행될 것으로 예측), 다른 현실적인 방법으로는 Hisory 테이블을 활용하여 이전의 branch 결과를 참고하여 예측하는 방법이 있음

3. 파이프라이닝에서 Branch Prediction의 역할은 무엇인가?
~~~
Branch Prediction은 Control Hazards를 위한 분기 결과를 예측함으로써 지연을 줄이려는 방법으로, 단순한 방법으로는 항상 분기 결과가 틀린 것(아닌 것)으로 가정하는 것이 있으며, 현실적인 방법으로는 history table을 이용하여 이전의 분기들을 참고해서 예측하는 Dynamic Branch Prediction이 있음
~~~

4. Branch Prediction에서 단순히 아닌 것으로 가정하는 방법을 사용했을 때의 문제점은? (굳이 Dynamic Branch Prediction을 사용하는 이유)
~~~
초대형(Stage가 아주 많은) Pipeline일수록, branch 패널티(stall)은 중요해짐(커짐)
이런 경우를 대비하여 Dynamic Branch Prediction을 활용하는 것이 적절함
~~~