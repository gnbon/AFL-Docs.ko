### 1) 커버리지 측정
---

컴파일시 프로그램에 삽입되는 Instrumentation은 분기문(edge) 커버리지와 분기문을 통과한 대략적인 횟수를 체크합니다. 분기문에 삽입되는 코드의 예시는 다음과 같습니다.

```C
  cur_location = <COMPILE_TIME_RANDOM>;
  shared_mem[cur_location ^ prev_location]++; 
  prev_location = cur_location >> 1;
```

cur_location 값은 컴파일 시 무작위로 생성됩니다. 이는 복잡한 project의 linking 과정을 간단하게 처리하고, XOR 연산 결과를 균일하게 분배하기 위함입니다.

shared_mem[] 배열은 afl-fuzzer에 의해 Instrumentation된 바이너리로 전달되는 64kB 크기의 공유메모리(SHM) 영역입니다. 공유 메모리 Bitmap의 모든 Byte set은 각각 Instrumentation 된 코드의 특정 (branch_src, branch_dst) 튜플에 대응되는 것으로 간주할 수 있습니다.

Map의 크기는 거의 모든 의도한 타겟과 일대일 대응되도록 선택됩니다. 이러한 타겟은 일반적으로 2000개에서 10000개의 분기점을 가지며, 다음은 그러한 범위 및 대응하는 타겟들의 예시입니다. 

```
   Branch cnt | Colliding tuples | Example targets
  ------------+------------------+-----------------
        1,000 | 0.75%            | giflib, lzo
        2,000 | 1.5%             | zlib, tar, xz
        5,000 | 3.5%             | libpng, libwebp
       10,000 | 7%               | libxml
       20,000 | 14%              | sqlite
       50,000 | 30%              | -
```

한편, 이러한 Map 사이즈들은 수신 측에서 밀리초(ms) 단위로 분석할수 있을 만큼 작은 크기이며, L2 캐시에 해당할 만큼 작은 크기입니다.

이러한 형태의 커버리지는 프로그램의 실행 경로에 대해 훨씬 더 많은 Insight를 제공합니다. 특히, 이는 세밀하게 다음과 같은 실행 Trace를 구별합니다.

```
  A -> B -> C -> D -> E (tuples: AB, BC, CD, DE)
  A -> B -> D -> C -> E (tuples: AB, BD, DC, CE)
```

이는 코드에서 미묘한 오류 조건을 발견하는 데 도움이 됩니다. 왜냐하면 보안 취약점들은 일반적인 새로운 블록에 도달하는 것에 연관되기보다, 예상치 못하거나 부적절한 상태 전환과 연관되기 때문입니다.

앞서 소개한 의사코드의 마지막 줄에서, 시프트 연산을 수행하는 이유는 튜플의 방향성을 보존하기 위함입니다(해당 연산이 없다면, A ^ B 는 B ^ A와 구별할 수 없을 것입니다). 또한 이는 tight 루프의 정체성을 보존하는 역할을 합니다(해당 연산이 없다면, A ^ A는 B ^ B와 동일할 것입니다).

Intel CPU에 포화 산술 연산 Opcode가 없음은 Hit counter가 0으로 Wrap around될  가능성이 존재함을 내포합니다. 그러나 이는 가능성이 상당히 낮으며 지엽적인 경우이므로, 성능 Trade-off로 수용하도록 간주됩니다.