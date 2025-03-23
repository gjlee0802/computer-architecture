# Performance

## 1. Response Time & Throughput (응답시간 & 처리량)
* **Response Time** (응답시간):
    * 하나의 operation을 수행하는데 걸리는 시간
* **Throughput** (처리량):
    * 단위 시간 당 얼마나 많은 일을 할 수 있는지

* Response Time과 Throughput은 무엇에 영향을 받는가?
    * 더욱 빠른 CPU로 교체하는 경우,
        * Response Time 감소
        * Throughtput 증가
    * CPU를 추가하는 경우,
        * Response Time 유지
        * Throughput 증가
    * 즉, Response Time은 Throughput에 영향을 주지만, Throughput은 Response Time에 영향을 주지 않을 수 있다.

## 2. Relative Performance (상대적 성능)
* 성능(Performance)비는 실행 시간(Execution Time)과의 역수관계
* 예를 들어,
    * A는 5s, B는 10s가 걸린다고 가정
    * Execution Time B / Execution Time A = 10s / 5s = 2
    * 따라서 A는 B보다 성능이 2배 빠름

## 3. Measuring Execution Time
* **Elapsed Time** (경과시간):
    * 모든 측면에서의 총 응답시간
    * 어떤 작업의 시작부터 끝까지 총 작업시간
    * 시스템 전체 Performance를 정의하는데 사용
* **CPU Time** (순수하게 CPU에서 걸리는 시간):
    * 주어진 일을 처리하는데 걸리는 시간: I/O time, other job's shares
    * User CPU Time과 System CPU Time으로 나뉨

## 4. CPU Clocking
* Clock Period: 
    * duration of a clock cycle(한 클럭 에 걸리는 시간)
    * rising edge 후 다음 rising edge까지의 길이
* Clock Frequency (= Clock Rate)
    * cycles per second(초당 싸이클 수)

* **Clock Cycle Time = 1 / Clock Rate**
    * 예를 들어,
        * Clock Frequency(Clock Rate)가 `3GHz`인 경우,
        * Clock Cycle Time = `1 / 3 x 10^9` = 0.33 ns
## 5. CPU Time
* CPU Time 
    * = `CPU Clock Cycles` x `Clock Cycle Time`
    * = `CPU Clock Cycles` / `Clock Rate`
### Example
~~~
컴퓨터 A: 2GHz Clock Rate, 10s CPU Time이라고 가정
컴퓨터 B: 6s CPU Time, Clock Cycles는 A의 1.2배라고 가정
컴퓨터 B를 컴퓨터 A보다 빠르게 설계하려고 할 때, 
컴퓨터 B의 Clock을 얼마나 빠르게 해야할까? (B의 Clock Rate)
~~~
* Clock Rate(B) 
    * = `Clock Cycles(B)` / `CPU Time(B)`
    * = `1.2` x `Clock Cycles(A)` / `6s`
* Clock Cycles(A)
    * = `CPU Time(A)` x `Clock Rate(A)`
    * = `10s` x `2GHz` = `20` x `10^9`
* Clock Rate(B)
    * = `1.2` x `20` x `10^9` / `6s`
    * = `24` x `10^9` / `6s` = `4GHz`

## 6. Instruction Count & CPI
* Instruction Count: 
    * 프로그램, ISA와 컴파일러 등에 의해 결정됨
* CPI (Cycles Per Instruction):
    * Instruction당 평균 cycle 수
    * CPU HW에 의해 결정됨

* Clock Cycles
    * = `Instruction Count` x `CPI(Cycles per Instruction)`
* CPU Time
    * = (`Instruction Count` x `CPI`) x `Clock Cycle Time`
    * = (`Instruction Count` x `CPI`) / `Clock Rate`

### Example
~~~
컴퓨터 A: Cycle Time = 250 ps, CPI = 2라고 가정
컴퓨터 b: Cycle Time = 500 ps, CPI = 1.2라고 가정

동일한 ISA일 경우, 어떤 컴퓨터가 얼마나 더 빠른가?
~~~
* CPU Time(A)
    * = `Instruction Count` x `CPI(A)` x `Clock Cycle Time(A)`
    * = `Instruction Count` x `2` x `250 ps`
    * = `Unstruction Count` x `500 ps`
* CPU Time(B)
    * = `Instruction Count` x `CPI(B)` x `Clock Cycle Time(B)`
    * = `Instruction Count` x `1.2` x `500 ps`
    * = `Instruction Count` x `600 ps`

* `CPU Time(B)` / `CPU Time(A)`
    * = 600 / 500 = 1.2
    * 따라서, 컴퓨터 A가 1.2배 더 빠름