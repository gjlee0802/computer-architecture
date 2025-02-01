# 면접 기출 예시
* 외부에서 수집한 자료, 본인이 임의로 만든 개념적인 질문을 포함함
* 대학원 면접을 대비하기 위한 자료임
* **서울대 컴퓨터공학부**는 2가지의 과목을 선택하여 면접을 보도록 되어있으며, 그림과 함께 심층적으로 묻는 문제가 출제되는 것으로 알고 있음 [심층 문제로 이동](https://github.com/gjlee0802/computer-architecture/blob/main/interview_questions/4-the-processor.md#pipeline-hazards-%EC%8B%AC%EC%B8%B5-%EB%AC%B8%EC%A0%9C)
* **카이스트 전산학부**는 컴퓨터 구조 뿐만 아니라 모든 과목에서 전반적인 범위에 걸쳐 질문을 하며, 개념 질문들, 핵심 개념을 위주로 출제되는 것으로 알고 있음 [개념 질문으로 이동](https://github.com/gjlee0802/computer-architecture/blob/main/interview_questions/4-the-processor.md#%EA%B0%9C%EB%85%90-%EC%A7%88%EB%AC%B8)

## 개념 질문
### ISA (Instruction set architecture)
* CISC와 RISC의 차이점은 무엇인가?
* Register File과 Cache의 차이는 무엇인가?
* Intel과 AMD가 만드는 CPU의 공통점과 차이점은 무엇인가?
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
- Structure Hazards
    - **정의**: 사용하고자 하는 하드웨어 리소스가 사용 중에 있어서 구조적으로 지연이 발생하는 문제
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


## Pipeline Hazards 심층 문제
### 1. 5단계의 Pipeline에서 아래의 코드가 수행된다고 하자  
~~~
add $3, $1, $2      ...(1)
and $12, $5, $3     ...(2)
or  $13, $3, $6     ...(3)
sub $14, $3, $$2    ...(4)
sw  $15, 100($3)    ...(5)
~~~
#### 1.1. 위 코드 수행과정에서 발생하는 hazard를 설명하시오
~~~
data hazard가 발생한다. 
(2)를 수행함에 있어서 EX 단계(ALU)에서 $3에 해당하는 값이 필요한데, 
add 명령어의 결과로 얻어지는 데이터이기 때문에 
Read After Write 데이터 의존성으로 인해 hazard가 발생한다.
~~~
#### 1.2. 위에서 발생한 hazard를 Forwarding 방법만으로 해결하려고 한다. 가능한지 여부를 답하시오. (1) Yes라면, Forwarding 과정을 구체적으로 설명하시오. (2) No라면, 그 이유를 설명하고 추가적으로 해결하는 방법을 설명하시오.
~~~
Yes, Forwarding 방법만으로 해결할 수 있다.
~~~
* (2)번 명령어의 경우, 
    * `$3`이 **EX 단계에서 필요**하므로, (1)번 명령어가 MEM 단계에 있을 때 **EX/MEM 레지스터의 값을 포워딩**하여 해결함
* (3)번 명령어의 경우,
    * (1)번 명령어가 WB 단계에 있을 때 **MEM/WB 레지스터로부터 데이터를 포워딩**할 수 있음


#### 1.3. 위에서 발생한 hazard를 stall 방법(bubble 추가)만으로 해결하려 한다. 위 코드를 모두 수행하기 위해 bubble이 총 몇 개 필요한지 설명하시오.

~~~
2개 필요하다. (2)번 명령어가 ID 단계에서 $3의 값을 읽으려면,
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
(2)와 (4) 사이에 Load-use hazard가 생긴다. 
(4)번 명령어는 $2의 값을 EX 단계에 피연산자로 사용해야 하는데, 
(4)번 명령어가 EX 단계에 들어섰을 때, 
(2)번 명령어는 WB 단계에 들어섰을 것이다.
따라서 (2)번 명령어는 WB 단계를 마치지 않아,
$2의 값을 얻을 수 없다.
~~~

#### 1.5. 새로 생긴 hazard를 Forwarding 방법 만으로 해결하려 한다. 가능한지 여부를 답하시오. (1) Yes라면, Forwarding 과정을 구체적으로 설명하시오. (2) No라면, 그 이유를 설명하고 추가로 해결하는 방법을 설명하시오
~~~
Yes, Forwarding 방법만으로 해결 가능하다.
이 문제는 RAW의 데이터 의존성에 의한 것이다.
(4)번 명령어가 EX 단계에 들어선 시점에,
(2)번 명령어는 MEM 단계를 수행하여 $2의 값을 메모리로부터 읽은 상태이다.
이때, $2의 값이 존재하는 상태이며, MEM/WB 레지스터에서 포워딩오면 해결이 가능하다.
~~~

* **이 문제에서 알 수 있는 Point**: Load-use hazard라고 해서 Forwarding으로 해결하지 못하는 것이 아님! 
    * Load-use hazard인데, 값이 존재하지 않는 경우가 Forwarding이 아닌, Code scheduling을 고려해야하는 것임

#### 1.6. 위에서 추가로 발생한 hazard를 stall 방법만으로 해력하려고 한다. bubble이 총 몇 개 필요한지 설명하시오.
~~~
bubble은 1개 필요하다. 
한번의 Stall이면 (4)번 명령에서 $2의 값이 필요한 시점(EX 단계)에,
(2)번 명령의 WB 단계를 마칠 수 있기 때문이다.
~~~