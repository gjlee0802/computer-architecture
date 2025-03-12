# 5.3. Measuring and Improving Cache Performance
* https://ydeer.tistory.com/146
* https://velog.io/@saewoohan/Ch-5-Large-and-Fast-Exploiting-Memory-Hierarchy

## 1. Measuring Cache Performance
* CPU time의 요소(종류)들(components)
    * program executiuon cycles
        * cache hit time을 포함
    * memory stall cycles
        * 주로 cache miss들로 부터 발생
* 간결화 가정(simplifying assumption)을 해보면,
![cache_performance_eq_1](./image_files/cache_performance_eq_1.png)

### (a). Cache Performance Exmaple
#### <문제에서 주어짐>
* I-cache miss rate = 2%
* D-cache miss rate = 4%
* Miss penalty = 100 cycles
* Base CPI (ideal cache) = 2
* Load & stores are 36% of instructions


#### <문제 풀이> 

**Step 1. [해석]:**  
(1) 캐시 미스율 (Cache Miss Rate)
* I-cache miss rate (명령어 캐시 미스율) = 2%
* D-cache miss rate (데이터 캐시 미스율) = 4%

(2) 미스 패널티 (Miss Penalty)
* Miss penalty = 100 cycles
    * 캐시 미스가 발생할 때마다, 데이터를 메모리에서 가져오는 데 추가로 100 사이클이 걸림

(3) 이상적인 CPI (Cycles Per Instruction)
* Base CPI (ideal cache) = 2
    * 즉, 캐시 미스가 전혀 발생하지 않는 이상적인 경우, 1개의 명령어를 실행하는 데 평균 2 사이클이 걸린다고 가정

(4) 로드 및 저장 명령어 비율
* Load & Store instructions = 36% (0.36)
    * 전체 명령어 중 36%는 메모리에서 데이터를 가져오거나 저장하는 명령어

**Step 2. [Instruction당 Miss Cycles 계산 과정]:**  
**Cache miss로 인해 추가되는 Cycle 수**를 계산하기!  
  
(1) I-cache(명령어 캐시)로 인한 추가 사이클
* **I-cache miss는 2% 확률(0.02)로 발생, 미스가 발생하면 100 사이클의 패널티가 존재**
* 따라서, 1개의 명령어당 평균적으로 cache miss로 인해 추가되는 사이클은...
    * 0.02 x 100 = 2
    * 즉, **I-cache 때문에 평균 2 사이클이 추가됨**

(2) D-cache(데이터 캐시)로 인한 추가 사이클
* 전체 명령어 중 36% (0.36)가 Load&Store 명령어
* **Load&Store 명령어가 실행될 때, D-cache miss는 4% 확률(0.04)로 발생, 미스가 발생하면 100 사이클의 패널티가 존재**
* 따라서, 1개의 명령어당 평균적으로 cache miss로 인해 추가되는 사이클은...
    * 0.36 x 0.04 x 100 = 1.44
    * 즉, **D-cache 때문에 평균 1.44 사이클이 추가됨**

(3) 계산 결과: Instrution당 Miss cycles
* I-cache Miss Cycle: 0.02 * 100 = 2
* D-cache Miss Cycle: 0.36 0.04 100 = 1.44

**Step 3. [Actual CPI(Cache Miss로 인한 실제 CPI) 계산 과정]:** 

Actual CPI는...
~~~
`Base CPI(이상 적인 경우)` + `I-cache 영향` + `D-cache 영향`
~~~

Actual CPI = 2 + (2 + 1.44) = 5.44
* Ideal CPU is 5.44/2 = 2.72 times faster
