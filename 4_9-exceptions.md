# 4.9. Exceptions
## 1. Exceptions and Interrupts
* 프로그램이 실행 중에 "**예상치 못한**" 상황 발생할 때, 이를 처리하기 위해 실행 흐름을 변경해야 함
    * 이때 발생하는 것이 예외(Exception)와 인터럽트(Interrupt)
    * 다른 ISA마다 용어(term)를 다르게 사용
        * ISA(Instruction Set Architecture, 명령어 집합 구조)마다 예외와 인터럽트를 다르게 정의하는 경우가 있음
        * 어떤 CPU는 예외도 인터럽트의 한 종류로 간주

* Exception(예외)은 CPU 내부에서 발생
    * e.g., undefined opcode, overflow, syscall, ...

* Interrupt(중단)은 외부 I/O Controller(CPU 외부의 장치, OS 등...)로부터 발생

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