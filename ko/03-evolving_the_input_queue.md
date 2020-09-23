### 3) Evolving the input queue
---

<!-- 
Mutated test cases that produced new state transitions within the program are
added to the input queue and used as a starting point for future rounds of
fuzzing. They supplement, but do not automatically replace, existing finds. 
-->

프로그램의 새로운 State 전이를 생성한 Mutated된 Test case는 Input queue에 추가되며, 이후 Fuzzing의 새로운 Round의 시작점으로 사용됩니다. 이 Case들은 기존 발견을 보완하는 역할을 하지만, 그것들을 자동적으로 대체하지는 않습니다.

<!--
In contrast to more greedy genetic algorithms, this approach allows the tool
to progressively explore various disjoint and possibly mutually incompatible
features of the underlying data format, as shown in this image:

  http://lcamtuf.coredump.cx/afl/afl_gzip.png
-->

보다 Greedy한 유전 알고리즘과는 다르게, 이 접근법을 통해 Fuzzer는 점진적으로 여러 갈래의 겹치지 않는 코드 커버리지들을 탐색할 수 있습니다. 동시에, 기반 데이터 포맷의 상호 양립할 수 없는 기능들을 탐색할 수 있습니다. 다음 이미지는 이러한 특징을 잘 나타내 줍니다.

  http://lcamtuf.coredump.cx/afl/afl_gzip.png

<!-- 
Several practical examples of the results of this algorithm are discussed
here:

  http://lcamtuf.blogspot.com/2014/11/pulling-jpegs-out-of-thin-air.html
  http://lcamtuf.blogspot.com/2014/11/afl-fuzz-nobody-expects-cdata-sections.html 
-->

알고리즘에 대한 몇 개의 실용적인 결과의 예시는 다음과 같습니다.
  
  http://lcamtuf.blogspot.com/2014/11/pulling-jpegs-out-of-thin-air.html
  http://lcamtuf.blogspot.com/2014/11/afl-fuzz-nobody-expects-cdata-sections.html

<!-- 
The synthetic corpus produced by this process is essentially a compact
collection of "hmm, this does something new!" input files, and can be used to
seed any other testing processes down the line (for example, to manually
stress-test resource-intensive desktop apps). 
-->

이 과정을 통해 생성된 인위적인 Corpus는 본질적으로 "hmm, this does something new!"라고 판단되는 Input file들을 간결하게 모은 것이며, 이는 미래에 다른 테스팅 과정의 Seed로 사용될 수 있습니다(예를 들어, 자원 집약적인 프로그램을 수동으로 Stress-testing할 수 있습니다).

<!-- 
With this approach, the queue for most targets grows to somewhere between 1k
and 10k entries; approximately 10-30% of this is attributable to the discovery
of new tuples, and the remainder is associated with changes in hit counts. 
-->

이 접근법으로, Target 프로그램의 Queue는 대부분 1000개~10000개의 항목으로 증가합니다. 이 중 대략 10~30%는 새로운 Tuple의 발견에 기인하며, 나머지는 Hit counts의 변화에 의한 것입니다.

<!-- 
The following table compares the relative ability to discover file syntax and
explore program states when using several different approaches to guided
fuzzing. The instrumented target was GNU patch 2.7.3 compiled with -O3 and
seeded with a dummy text file; the session consisted of a single pass over the
input queue with afl-fuzz:

    Fuzzer guidance | Blocks  | Edges   | Edge hit | Highest-coverage
      strategy used | reached | reached | cnt var  | test case generated
  ------------------+---------+---------+----------+---------------------------
     (Initial file) | 156     | 163     | 1.00     | (none)
                    |         |         |          |
    Blind fuzzing S | 182     | 205     | 2.23     | First 2 B of RCS diff
    Blind fuzzing L | 228     | 265     | 2.23     | First 4 B of -c mode diff
     Block coverage | 855     | 1,130   | 1.57     | Almost-valid RCS diff
      Edge coverage | 1,452   | 2,070   | 2.18     | One-chunk -c mode diff
          AFL model | 1,765   | 2,597   | 4.99     | Four-chunk -c mode diff 
-->

다음은 Guided fuzzing의 여러가지 다른 접근법을 사용할 때, 파일 문법을 탐색하는 능력과 프로그램 상태를 탐색하는 능력을 비교한 표입니다. Instrumentation된 Target 프로그램은 -O3로 컴파일된 GNU patch 2.7.3이며, 더미 텍스트 파일을 시드로 설정하였습니다. Testing 과정은 afl-fuzz의 Input Queue를 1회 통과하는 것으로 구성되었습니다.

```
    Fuzzer guidance | Blocks  | Edges   | Edge hit | Highest-coverage
      strategy used | reached | reached | cnt var  | test case generated
  ------------------+---------+---------+----------+---------------------------
     (Initial file) | 156     | 163     | 1.00     | (none)
                    |         |         |          |
    Blind fuzzing S | 182     | 205     | 2.23     | First 2 B of RCS diff
    Blind fuzzing L | 228     | 265     | 2.23     | First 4 B of -c mode diff
     Block coverage | 855     | 1,130   | 1.57     | Almost-valid RCS diff
      Edge coverage | 1,452   | 2,070   | 2.18     | One-chunk -c mode diff
          AFL model | 1,765   | 2,597   | 4.99     | Four-chunk -c mode diff
```

<!-- 
The first entry for blind fuzzing ("S") corresponds to executing just a single
round of testing; the second set of figures ("L") shows the fuzzer running in a
loop for a number of execution cycles comparable with that of the instrumented
runs, which required more time to fully process the growing queue. 
-->

첫번째 Blind fuzzing 항목("S")은 단 한번만 테스팅한 결과이며, 두번째 Blind fuzzing 항목("L")은 여러 번의 실행 Cycle에 대한 루프를 실행한 결과를 보여줍니다. 이는 증가하는 Queue 전체를 실행하기 위해 더 많은 시간이 요구되는 Instrumentation에 대한 결과입니다.

<!-- 
Roughly similar results have been obtained in a separate experiment where the
fuzzer was modified to compile out all the random fuzzing stages and leave just
a series of rudimentary, sequential operations such as walking bit flips.
Because this mode would be incapable of altering the size of the input file,
the sessions were seeded with a valid unified diff:

    Queue extension | Blocks  | Edges   | Edge hit | Number of unique
      strategy used | reached | reached | cnt var  | crashes found
  ------------------+---------+---------+----------+------------------
     (Initial file) | 624     | 717     | 1.00     | -
                    |         |         |          |
      Blind fuzzing | 1,101   | 1,409   | 1.60     | 0
     Block coverage | 1,255   | 1,649   | 1.48     | 0
      Edge coverage | 1,259   | 1,734   | 1.72     | 0
          AFL model | 1,452   | 2,040   | 3.16     | 1 
-->

모든 무작위한 Fuzzing 단계를 거친 뒤 일련의 기본적이며 순차적인 Fuzzing 단계(Walking bit flip 등)를 수정한 각각의 실험에서 비슷한 결과를 확인할 수 있었습니다. 해당 모드에서는 Input file의 크기를 변경할 수 없기 때문에, 이 실험에서는 규격화된 유효한 diff 지점만을 변경하며 Seed로 사용하였습니다. 

```
    Queue extension | Blocks  | Edges   | Edge hit | Number of unique
      strategy used | reached | reached | cnt var  | crashes found
  ------------------+---------+---------+----------+------------------
     (Initial file) | 624     | 717     | 1.00     | -
                    |         |         |          |
      Blind fuzzing | 1,101   | 1,409   | 1.60     | 0
     Block coverage | 1,255   | 1,649   | 1.48     | 0
      Edge coverage | 1,259   | 1,734   | 1.72     | 0
          AFL model | 1,452   | 2,040   | 3.16     | 1
```

<!-- 
At noted earlier on, some of the prior work on genetic fuzzing relied on
maintaining a single test case and evolving it to maximize coverage. At least
in the tests described above, this "greedy" approach appears to confer no
substantial benefits over blind fuzzing strategies. 
-->

앞서 언급했듯이, Genetic fuzzing의 선행 연구들은 단일 Test case를 유지하고 이를 커버리지를 극대화하도록 진화시키는 방식을 사용하였습니다. 위의 실험에서 밝혀졌듯이, 이러한 "탐욕법적" 접근은 Blind fuzzing 전략보다 덜 효과적인 것으로 보여집니다.














