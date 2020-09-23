### 4) Culling the corpus
---
<!-- 
The progressive state exploration approach outlined above means that some of the test cases synthesized later on in the game may have edge coverage that is a strict superset of the coverage provided by their ancestors.
-->

위에서 설명한 진보된 상태 탐색 접근법은 게임에서 나중에 합쳐진 테스트케이스에 몇가지 Edge 커버리지가 존재할 수 있다는 것을 의미합니다. Edge 커버리지는 그들의 Ancestor들이 제공한 커버리지의 엄격한 Superset(strict superset)입니다.


<!-- 
To optimize the fuzzing effort, AFL periodically re-evaluates the queue using a
fast algorithm that selects a smaller subset of test cases that still cover
every tuple seen so far, and whose characteristics make them particularly
favorable to the tool. 
-->

퍼징을 최적화하기 위해서, AFL은 빠른 알고리즘을 이용하여 주기적으로 큐를 재평가 합니다. 이 빠른 알고리즘은 지금까지 본 모든 튜플을 계속 커버(still cover)하고 있는 테스트 케이스들의 작은 하위 집합을 하나 선택하는 알고리즘입니다. 알고리즘의 이러한 특성은 이 Tool에 특히 더 유리합니다.

<!-- 
The algorithm works by assigning every queue entry a score proportional to its
execution latency and file size; and then selecting lowest-scoring candidates
for each tuple. 
-->

이 알고리즘은 모든 큐 항목들에 항목들의 실행 지연시간과 파일 크기에 비례한 점수를 할당합니다. 그리고 각 튜플에서 제일 낮은 점수를 가지는 항목들을 고릅니다.

<!-- 
The tuples are then processed sequentially using a simple workflow:

  1) Find next tuple not yet in the temporary working set,

  2) Locate the winning queue entry for this tuple,

  3) Register *all* tuples present in that entry's trace in the working set,

  4) Go to #1 if there are any missing tuples in the set. 
-->

그러고 나면, 튜플들은 단순한 Workflow를 따라 순차적으로 실행됩니다. 

  1) 아직 임시 작업 세트에 없는 다음 튜플을 찾고 
  
  2) 이 튜플에 대한 Winning 큐 항목을 탐색합니다.
  
  3) 작업 세트안 항목의 Trace에 존재하는 *모든* 튜플들을 기록합니다.
  
  4) 만약 세트안에 놓친 튜플들이 있다면 #1로 돌아갑니다.


<!-- 
The generated corpus of "favored" entries is usually 5-10x smaller than the starting data set. Non-favored entries are not discarded, but they are skipped
with varying probabilities when encountered in the queue: 
-->

생성된 "favored" 항목들의 Corpus는 보통 처음에 시작했던 Data set보다 5~10배 정도는 작습니다. Non-favored 항목들은 버려지지는 않지만, 큐안에서 마주쳤을때 다양한 확률들로 스킵됩니다. :

<!-- 
    - If there are new, yet-to-be-fuzzed favorites present in the queue, 99%
    of non-favored entries will be skipped to get to the favored ones.

  - If there are no new favorites:

    - If the current non-favored entry was fuzzed before, it will be skipped
      95% of the time.

    - If it hasn't gone through any fuzzing rounds yet, the odds of skipping
      drop down to 75%.
 -->

  - 만약에 '새로운 것'(큐안에 아직 퍼징되지 않은 favorite들)이 있다면, Non-favored 항목의 99%는 favored 한 것들에 도달하기 위해 스킵될 것입니다.

  - 만약 새로운 favorite들이 존재하지 않는 다면 :

    - 만약 최근에 Non-favored 항목들이 퍼징됐다면, 95%의 시간이 스킵됩니다.

    - 만약에 그게 아직 어떠한 퍼징 라운드도 거치지 않았다면, 스킵할 가능성은 75%로 떨어집니다.

<!-- 
Based on empirical testing, this provides a reasonable balance between queue cycling speed and test case diversity. 
-->

지금까지 테스트해본 결과에 의하면, 이 알고리즘은 큐 Cycling speed와 테스트 케이스의 다양성 사이에 균형을 합리적으로 잡아줍니다.

<!-- 
Slightly more sophisticated but much slower culling can be performed on input or output corpora with afl-cmin. This tool permanently discards the redundant entries and produces a smaller corpus suitable for use with afl-fuzz or external tools. 
-->

afl-cmin을 사용해서, 입출력 Corpora에 약간 더 정교하긴하지만 훨씬 더 느린 탐색(culling)을 수행할 수 있습니다. 이 도구는 중복 항목들을 영구적으로 버리고 afl-fuzz나 다른 도구들의 사용에 좀 더 적합한 더 작은 Corpus를 생산합니다.