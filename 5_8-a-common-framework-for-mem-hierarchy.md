# 5.8. A Common Framework for Memory Hierarchy
## 1. The Memory Hierarchy
~~~
1. Block Placement (블록 배치)
2. Finding a Block (데이터 찾기)
3. Replacement on a Miss (캐시 미스 시 교체 정책)
4. Write Policy (쓰기 정책)
~~~
* common principle이 메모리 계층의 모든 단계에 적용된다.
    * caching의 개념이 동일하게 적용됨
    * 즉, 빠른 메모리에 자주 사용하는 데이터를 저장해서 성능을 높이자는 원칙
* 각 레벨의 hierarchy에서
    * **Block Placement (블록 배치)**: 메모리 계층에서 특정 데이터를 어디에 저장할지 결정하는 규칙
        * **Associativity가 높을수록**:
            * 장점: **Miss rate가 감소**
            * 단점: **복잡성, 접근 시간, 비용 증가**
        * Associativity에 따른 방법:  
            * Direct Mapped Cache: 특정 주소가 오직 한 곳에만 저장
            * Set Associative Cache: 여러 캐시 라인을 하나의 Set으로 묶어 저장
            * Fully Associative Cache: 모든 캐시 라인 중 아무 곳에나 저장 가능
    * **Finding a Block (데이터 찾기)**: 저장된 데이터를 빠르게 찾는 방법
        * Tag를 이용한 검색이 일반적
            * PIPT: Physical Address를 Cache Tag로 사용 => TLB 주소 변환 먼저 수행
            * VIVT: Virtual Address를 Cache Tag로 사용 => Aliasing(여러 VA가 같은 PA 가리키는) 문제
            * VIPT: Virtual Address의 일부를 Index, Physical Address를 기반으로 한 Tag 사용
        * TLB: 가상 주소 => 물리 주소 변환 빠르게 수행
        * Cache: Tag를 비교해서 Cache Hit 여부 확인
    * **Replacement on a Miss (캐시 미스 시 교체 정책)**: 캐시에 데이터가 없다면(Cache Miss), 기존 데이터를 어떤 기준으로 교체할 것인지 결정
        * LRU: 가장 오랫동안 사용하지 않은 데이터를 교체
        * FIFO: 가장 먼저 들어온 데이터를 교체
        * Random: 무작위로 교체
    * **Write Policy (쓰기 정책)**: 캐시에 데이터를 저장할 때, 메인 메모리(RAM)와의 동기화 방법을 결정
        * Write-Through: 캐시와 메인 메모리 동시 반영
        * Write-Back: 데이터를 캐시(Upper level)에만 먼저 저장, 나중에 메모리(Lower level)에 반영
            * dirty bit가 필요함
        * **Virtual Memory는 디스크 쓰기 지연 때문에 Write-Back만 가능함!**

## 2. Block Placement (블록 배치)
* **Block Placement (블록 배치)**: 메모리 계층에서 특정 데이터를 어디에 저장할지 결정하는 규칙
    * **Associativity가 높을수록**:
        * 장점: **Miss rate가 감소**
        * 단점: **복잡성, 접근 시간, 비용 증가**
    * Associativity에 따른 방법:  
        1. Direct Mapped Cache: 특정 주소가 오직 한 곳에만 저장
        2. Set Associative Cache: 여러 캐시 라인을 하나의 Set으로 묶어 저장
        3. Fully Associative Cache: 모든 캐시 라인 중 아무 곳에나 저장 가능

## 3. Finding a Block (데이터 찾기)
* **Finding a Block (데이터 찾기)**: 저장된 데이터를 빠르게 찾는 방법
    * Tag를 이용한 검색이 일반적
        1. PIPT: Physical Address를 Cache Tag로 사용 => TLB 주소 변환 먼저 수행
        2. VIVT: Virtual Address를 Cache Tag로 사용 => Aliasing(여러 VA가 같은 PA 가리키는) 문제
        3. VIPT: Virtual Address의 일부를 Index, Physical Address를 기반으로 한 Tag 사용
    * TLB: 가상 주소 => 물리 주소 변환 빠르게 수행
    * Cache: Tag를 비교해서 Cache Hit 여부 확인
## 4. Replacement on a Miss (Cache Miss 시 교체 정책)
* **Replacement on a Miss (캐시 미스 시 교체 정책)**: 캐시에 데이터가 없다면(Cache Miss), 기존 데이터를 어떤 기준으로 교체할 것인지 결정
    1. LRU: 가장 오랫동안 사용하지 않은 데이터를 교체
    2. FIFO: 가장 먼저 들어온 데이터를 교체
    3. Random: 무작위로 교체

## 5. Write Policy (쓰기 정책)
* **Write Policy (쓰기 정책)**: 캐시에 데이터를 저장할 때, 메인 메모리(RAM)와의 동기화 방법을 결정
    * Write Policy 2 가지:
        1. Write-Through: 캐시와 메인 메모리 동시 반영
        2. Write-Back: 데이터를 캐시(Upper level)에만 먼저 저장, 나중에 메모리(Lower level)에 반영
            * dirty bit가 필요함
    * **Virtual Memory는 디스크 쓰기 지연 때문에 Write-Back만 가능함!**

## 6. Sources of Misses
* **필수 미스: Compulsory misses (aka. Cold start misses)**
    * 원인: **block에 처음 접근할 때, 데이터가 없기에** 모두 miss 발생
    * 해결 방법:
        * 블록 크기 증가 => (단, 전체 Miss rate 증가)
* **용량 미스: Capacity miss**
    * 원인: **캐시 크기가 부족해서 기존의 블록이 강제로 제거**됨
    * 해결 방법:
        * 캐시 크기 증가
        * 데이터를 작은 블록으로 나누어 효율적으로 캐싱
* **충돌 미스: Conflict misses (aka. Collision misses)**
    * 원인: **set 내부에서의 entry끼리 경쟁으로 발생**
    * 해결 방법:
        * Set의 개수 증가
        * Fully Associative 방식 사용
    * non-fully associative cache에서만 발생함
        * 이유: 충분히 들어갈 공간이 있음에도 특정 Set이 가득 차서 Cache Miss가 발생 (특정 주소가 특정 Cache Set에만 저장)
            * Fully Associative Cache에서는 어느 캐시 Set에도 저장 가능하므로, 특정 Set에 국한되지 않음
    * Fully associative cache에서는 발생하지 않음

## 7. Cache 설계의 Trade-offs
| 설계 변경 | 미스율에 미치는 영향 | 부정적 성능 영향 |
|-----------|---------------------|------------------|
| **캐시 크기 증가** | 용량 미스(Capacity Miss) 감소 | 접근 시간 증가 가능 |
| **연관도(Associativity) 증가** | 충돌 미스(Conflict Miss) 감소 | 접근 시간 증가 가능 |
| **블록 크기 증가** | 필수 미스(Compulsory Miss, Cold Start Miss) 감소 | 미스 패널티 증가. 블록 크기가 너무 크면 오염(Pollution)으로 인해 미스율 증가 가능 |
