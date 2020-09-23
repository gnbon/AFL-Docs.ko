### 9) Investigating crashes
---
<!-- 
The exploitability of many types of crashes can be ambiguous; afl-fuzz tries to address this by providing a crash exploration mode where a known-faulting test case is fuzzed in a manner very similar to the normal operation of the fuzzer, but with a constraint that causes any non-crashing mutations to be thrown away.
-->


Crash들의 다양한 유형의 익스가능성은 모호할 수 있습니다. Afl-fuzz는 Crash 탐색 모드를 제공하여 이 문제를 해결하려고 노력합니다. Crash 탐색모드는 알려진 결함이 있는 테스트 케이스를 일반 퍼저의 작동과 매우 유사한 방법으로 퍼징하는 모드입니다.근데 이 모드에는 Crash를 발생시키지 않는 뮤테이션들을 버리는 제약조건이 있습니다.

<!-- 
A detailed discussion of the value of this approach can be found here: 

http://lcamtuf.blogspot.com/2014/11/afl-fuzz-crash-exploration-mode.html
-->

이 접근법의 값에 대한 자세한 내용은 여기서 확인할 수 있습니다.

  http://lcamtuf.blogspot.com/2014/11/afl-fuzz-crash-exploration-mode.html

<!-- 
The method uses instrumentation feedback to explore the state of the crashing program to get past the ambiguous faulting condition and then isolate the newly-found inputs for human review.
 -->

이 방법은 모호한 결함 상태를 지나치게 하기위해서, Instrumentation Feedback을 사용해서 Crashing 프로그램의 상태를 탐색 합니다. 그리고 새롭게 찾은 인풋들을 사람이 보기 편하게 하기 위해 따로 분리해둡니다.

<!-- 
On the subject of crashes, it is worth noting that in contrast to normal queue entries, crashing inputs are *not* trimmed; they are kept exactly as discovered to make it easier to compare them to the parent, non-crashing entry in the queue. That said, afl-tmin can be used to shrink them at will.
 -->

Crash에 관해서, 일반적 큐 항목들과는 달리 요부분은 주목할 가치가 있습니다. Crashing 인풋들은 다듬여 잘려나가지 *않습니다.* 그들은 발견된 그대로 유지됩니다. 그래야 쉽게 큐에 있는 Crash를 유발하지 않은 상위(parent) 항목들과 쉽게 비교할 수 있기 때문입니다.