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
    
## 3. An Alternate Mechanism