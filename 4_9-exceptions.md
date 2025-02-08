# 4.9. Exceptions
## 1. Exceptions and Interrupts
* 프로그램이 실행 중에 "**예상치 못한**" 상황 발생할 때, 이를 처리하기 위해 실행 흐름을 변경해야 함
    * 이때 발생하는 것이 예외(Exception)와 인터럽트(Interrupt)
    * 다른 ISA마다 용어(term)를 다르게 사용
        * ISA(Instruction Set Architecture, 명령어 집합 구조)마다 예외와 인터럽트를 다르게 정의하는 경우가 있음
        * 어떤 CPU는 예외도 인터럽트의 한 종류로 간주

* Exception(예외)은 CPU 내부에서 발생
    * e.g., undefined opcode, overflow, syscall, ...

* Interrupt(인터럽트)는 외부 I/O Controller(CPU 외부의 장치, OS 등...)로부터 발생

* Exception vs. Interrupt
    | 구분 | 예외 (Exception) | 인터럽트 (Interrupt) |
    |------|----------------|----------------|
    | 발생 원인 | 프로그램 내부 문제 (0 나누기, 메모리 접근 오류 등) | 외부 장치 이벤트 (키보드 입력, 네트워크 요청 등) |
    | 발생 시점 | 명령어 실행 중 | 명령어 실행과 무관하게 언제든지 발생 |
    | 처리 방식 | CPU가 예외 핸들러(Exception Handler) 호출 | CPU가 인터럽트 핸들러(Interrupt Handler) 호출 |



* Trap이란?
    * Software Interrupt라고도 불림
    * syscall 활용하여 명령어가 Interrupt를 발생시킴
    * 원래 Interrupt는 HW에서 발생시키지만, SW적으로 호출해서 발생시키는 경우

* 이 문제를 다루기 위해서는 어느정도의 성능 저하를 감수해야 함

## 2. Handling Exceptions
* MIPS에서, Exception은 System Control Coprecessor(CP0)에 의해 관리됨
* 문제가 생긴(offending, interrupted) Instruction의 PC(즉, 현재 Instruction 주소)를 저장함
    * MIPS에서는 문제 생긴 PC값을 **EPC(Exception Program Counter)에 저장**
* 문제의 종류(indication. 표시,원인)를 저장함
    * MIPS에서는 문제의 종류를 **Cause Register에 저장**
    * 예를 들어, 1-bit로 가정한다면,
        * undefined opcode는 0, overflow는 1로 저장하는 방식
        * 실제로는 여러 문제 종류가 존재
* 위의 EPC, Cause Register 등에 저장 후에는
    * 미리 정해진(8000 00180) 메모리 주소에 위치한 handler로 jump하는데,
    * 여기서 EPC와 Cause Register의 값 등을 이용하여 처리하게 됨

## 3. An Alternate Mechanism - 다른 방식(좀 더 발전된 방식) 알아보기
* Vectored Interrupts
    * 원인에 따른 handler 주소가 다르게 정해져 있음
    * 즉, 미리 정해진 메모리 주소의 handler로 jump하는 것이 아니라, 각각의 원인에 handler 주소가 정해짐
    * 예시:
        * undefined opcode:　C000 0000
        * overflow:　　　　　 C000 0020
        * ...:　　　　　　　 　 C000 0040
* 문제가 발생한 Instruction은 다음 중 하나로 진행됨
    * (드물게) 문제가 발생했을 때, 예외(또는 인터럽트) 핸들러가 즉시 문제를 해결하는 경우
        * 그러나, 대부분의 경우에는 발생한 문제를 해결하려면 더 복잡한 처리가 필요함
    * 처음 실행된 핸들러가 "**이 문제를 제대로 해결할 수 있는 Real Handler**"로 이동(jump)해서 처리하는 경우

## 4. Handler Actions
* 원인을 읽고, 관련된 handler로 이동(jump)
* handler는 필요한 동작을 결정함
* 만약, **해당 명령어를 재시작** 가능하다면
    * 적절한 동작으로 시정하여 동작시킴
    * **EPC를 이용하여 원래 실행 중이던 명령어(원래의 PC)로 돌아갈 수 있도록 함**
* 그렇지 않다면
    * 프로그램을 중단(terminate)
    * EPC, cause, ... 등을 이용하여 error를 보고(report error)

## 5. Exceptions in a Pipeline
* Control Hazard의 또 다른 형태
    * Control Hazard는 프로그램의 flow가 바뀔지도 몰라 직후 명령어가 바로 실행될 수 없는 hazard를 말함
    * Exception도 발생하면, 다음의 명령어들이 실행되지 못하고 Exception을 처리해야 함
* 아래 add 명령어의 EX Stage에서 Overflow가 발생했다고 생각해봤을 때, Exception 처리 과정은 아래와 같음
    ~~~
    add $1, $2, $1
    ~~~
    * `$1`에 잘못된 값이 쓰여지는 것을 막음
    * 이전 Instruction들은 제대로 완수될 수 있도록 함
    * add 명령어와 이후 명령어들을 비움 (Flush)
    * **EPC와 Cause Register의 값 세팅**
    * **handler로** 제어권을 넘김 (handler가 처리할 수 있도록 jump)
* 예측 실패한 branch와 유사함!
    * **프로그램 흐름(Control Flow)이 변경됨**
        * 분기 예측 실패
            * 예측된 분기 방향에서 잘못된 예측임을 깨닫게 되면 올바른 분기 방향으로 되돌아감
            * 명령어를 취소(Flush)하고 올바른 명령어를 실행
        * Exception
            * Exception이 발생하면 이후 명령어를 실행할 수 없음
            * 이후 명령어들을 취소(Flush)하고, Exception Handler로 jump
    * **파이프라인에서 명령어를 취소(Flush)하는 과정이 동일**
    * **동일한 HW 자원을 사용**
        * branch 예측 실패와 예외 발생 모두, 현재 실행 중인 명령어의 실행을 멈추고 새로운 PC(주소)를 설정해야 함
            * 이를 위해 EPC와 같은 레지스터를 사용하여 새로운 명령어 실행을 준비

## 6. Pipeline with Exceptions
![pipeline_with_exceptions](./image_files/pipeline_with_exceptions.png)
* 추가된 부분을 주목할 것  
    * Flush 관련 Control 신호: ID.Flush, EX.Flush
    * EPC와 Cause Register
* 추가된 장치 외에도, 기존의 장치를 많은 부분 함께 활용하여 exception을 처리  

## 7. Exception Properties(특성)
* 재실행 가능한 명령어에 대한 Exception 처리 과정
    * Pipeline은 명령어를 비움 (Flush)
    * Handler가 처리 후, 원래의 instruction으로 돌아감
    * 이번엔 시정된 instruction을 실행함
* EPC 레지스터에 PC가 저장됨
    * 문제가 발생한 instruction을 식별가능
    * 실제론 PC+4가 저장될 것
    * Handler가 반드시 조정해야 함(-4를 PC에 넣어줘야 해당 instruction으로 돌아감)


## 8. Exception 예제
아래의 예시에서 **add 명령어에서 exception 발생**했다고 가정  
  
40  sub $11, $2, $4  
44  and $12, $2, $5  
48  or  $13, $2, $6  
**4C  add $1, $2, $1**  
50  slt $15, $6, $7  
54  lw  $16, 50($7)  
...  
  
이를 처리하는 **handler**는 아래와 같다고 가정  
  
Handler  
8000 0180   sw $25, 1000($0)  
8000 0184   sw $26, 1004($0)  
...  
  
### EX stage에서 exception 발생
현재 instruction(add)과 이후 instruction들 모두 flush  
![exception_example_1](./image_files/exception_example_1.png)  

### nop으로 flush(bubble), 그리고 handler로 jump
수행 중이던 instruction들은 flush 되며, IF stage에 handler 명령어 수행됨
![exception_example_2](./image_files/exception_example_2.png)  
  
## 9. Multiple Exceptions
* Pipeline은 여러 명령어를 동시에 수행
    * 그러므로 동시에 여러 exception이 발생할 수도 있음
* 간단한 접근법: 더 일찍인(가장 진척도가 높은, 가장 후의 stage에서 수행 중인) instruction의 예외부터 처리
* 뒤이어 오던 instruction들은 함께 Flush
* Multiple Exceptions을 처리할 때, 가장 먼저 실행된(진척도가 가장 높은) 명령어의 예외부터 처리하면, **Precise Exception의 원칙을 유지**할 수 있음
    * 즉, "Precise Exception"을 유지하기 위한 방법으로 더 앞선 명령어의 예외를 먼저 처리하고, 뒤따르는 명령어들을 flush하는 방식이 제안된 것
        * ✅ 앞선 명령어까지는 정상적으로 완료됨
        * ✅ 예외가 발생한 시점에서 모든 이후 명령어는 사라짐
        * ✅ 즉, Precise Exception의 원칙을 만족함
* 복잡한(complex) pipeline에서는
    * Cycle마다 여러 명령어를 실행할 수 있음
    * 명령어의 결과 순서가 실행 순서와 같지 않음(out-of-order)
    * 규칙(명령어 순서대로, Precise exception 원칙)을 지키기가 어려움
### "Precise" Exception이란?
* Precise Exception은 예외가 발생했을 때 프로그램이 정확하게 정의된 상태(consistent state) 를 유지하도록 하는 개념
* 이를 만족하려면:
    * 예외가 발생하기 전까지의 명령어들은 모두 정상적으로 커밋(완료)되어야 함
    * 예외가 발생한 명령어 이후의 모든 명령어들은 수행되지 않은 상태여야 함 (즉, flush 처리)

## 10. Imprecise Exceptions
### Imprecise Exception이란? 
* 예외가 발생했을 때 CPU의 상태가 정확하게 정리되지 않는 경우를 말함
* 즉, 어떤 명령어가 실행되었고, 어떤 명령어가 실행되지 않았는지 명확하지 않을 수 있는 상태가 되는 것을 말함

### Precise Exception vs. Imprecise Exception 비교
| **구분**             | **Precise Exception** | **Imprecise Exception** |
|---------------------|------------------|------------------|
| **정확한 상태 유지** | ✅ 유지됨 (예외 발생 이전의 상태만 남음) | ❌ 불명확 (예외 발생 후 일부 명령어가 실행되었을 수도 있음) |
| **파이프라인 처리**  | 예외 발생한 명령어 이전까지 실행, 이후는 모두 flush | 파이프라인을 멈추고, 현재 상태 그대로 저장 |
| **복구 방식**        | 간단 (예외 발생 직후 상태가 일관됨) | 복잡 (소프트웨어가 분석 후 처리 필요) |
| **사용 가능 환경**   | Out-of-Order 실행에서도 가능 | 단순한 In-Order CPU에서만 가능 |

### Imprecise 처리 방식
1. **CPU가 실행을 멈추고, 현재 상태를 저장**
    * 실행 중이던 모든 명령어의 진행 상황과 예외 원인을 저장
2. **소프트웨어 핸들러(handler)가 개입하여 예외 처리**
    * 어느 명령어(하나 혹은 여러 개)가 예외를 발생시켰는지 파악
    * 어떤 명령어를 실행 완료해야 할지(Manual Completion), 어떤 명령어를 무효화(Flush)할지 결정
3. **하드웨어는 단순화되지만, 소프트웨어 핸들러가 복잡해짐**
    * 예외 발생 시 하드웨어가 즉시 처리하지 않고, 예외 정보를 저장한 후 소프트웨어적으로 분석 및 처리
    * 장점: 하드웨어 설계를 단순화할 수 있음
    * 단점: 소프트웨어 핸들러의 부담 증가, 복구 과정이 복잡해질 수 있음
4. **Out-of-Order 실행을 지원하는 최신 CPU에서는 사용이 어려움**
    * 최신 CPU는 명령어가 순서대로 실행되지 않음(Out-of-Order Execution)
    * 따라서 예외 발생 시 어떤 명령어가 완료되었고, 어떤 명령어가 완료되지 않았는지 알기 어려움
    * 즉, Imprecise Exception을 적용하면 CPU 상태가 불확실해지는 문제가 발생 → **최신 CPU에서는 사용 불가능**

