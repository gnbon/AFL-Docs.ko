### 5) Trimming input files
---

<!--
File size has a dramatic impact on fuzzing performance, both because large
files make the target binary slower, and because they reduce the likelihood
that a mutation would touch important format control structures, rather than redundant data blocks. This is discussed in more detail in perf_tips.txt.
-->

파일의 크기는 Fuzzing 성능에 큰 영향을 미칩니다. 그 이유는 큰 파일은 Target binary를 느리게 만들 뿐 아니라, 파일이 커질 수록 덜 중요한 데이터를 Mutation할 가능성이 높아지며, 중요한 형식 제어 구조를 Mutation할 가능성은 낮아지기 때문입니다.

<!--
The possibility that the user will provide a low-quality starting corpus aside,
some types of mutations can have the effect of iteratively increasing the size
of the generated files, so it is important to counter this trend.
-->

사용자가 저품질의 Corpus를 사용할 가능성은 논외로 하고, 일부 유형의 Mutation은 생성되는 파일의 크기를 반복적으로 증가시키는 효과를 낼 수 있으므로, 이러한 동향에 대응해서 Corpus를 사용해야 합니다.

<!--
Luckily, the instrumentation feedback provides a simple way to automatically
trim down input files while ensuring that the changes made to the files have no
impact on the execution path. 
-->

다행히도, Instrumentation 피드백은 입력 파일을 자동으로 Trimming하는 간단한 방법을 제공하는 동시에 파일의 변경 사항이 Execution Path에 영향을 미치지 않도록 보장합니다.

<!--
The built-in trimmer in afl-fuzz attempts to sequentially remove blocks of data
with variable length and stepover; any deletion that doesn't affect the checksum
of the trace map is committed to disk. The trimmer is not designed to be
particularly thorough; instead, it tries to strike a balance between precision
and the number of execve() calls spent on the process, selecting the block size
and stepover to match. The average per-file gains are around 5-20%.
-->

afl-fuzz에 내장된 Trimmer는 순차적으로 가변 길이 혹은 잘못 짚었다고 추측되는(Stepover) 데이터 블록들을 제거합니다. Trace map의 Checksum에 영향을 미치지 않는 모든 삭제는 디스크에 저장됩니다. Trimmer는 특별히 철저하게 설계되지 않았으나, 대신 정확성과 execve() 함수 호출 사이의 균형을 맞추려 하고 이에 맞는 블록 크기와 Stepover를 선택합니다. 파일 당 평균 이득은 약 5~20% 사이입니다.

<!--
The standalone afl-tmin tool uses a more exhaustive, iterative algorithm, and
also attempts to perform alphabet normalization on the trimmed files. The
operation of afl-tmin is as follows.
-->

독립적인 afl-tmin은 보다 철저하고 반복적인 알고리즘을 사용하며, Trimming된 파일에 대해 문자 간략화를 시도합니다. afl-tmin의 동작은 다음과 같습니다.

<!--
First, the tool automatically selects the operating mode. If the initial input
crashes the target binary, afl-tmin will run in non-instrumented mode, simply
keeping any tweaks that produce a simpler file but still crash the target. If
the target is non-crashing, the tool uses an instrumented mode and keeps only
the tweaks that produce exactly the same execution path.
-->

먼저 afl-tmin은 동작 모드를 자동으로 선택합니다. 초기 Input이 Target binary에서 Crash를 내면, afl-tmin은 Non-instrumented 모드로 실행되어 여전히 Crash를 유발하는 Tweak들을 유지하지만 간소화된 파일을 생성합니다. 만약 Target이 Crash되지 않으면, afl-tmin은 Instrumented 모드로 실행되어 동일한 Execution path를 유발하는 Tweak들만을 가지도록 간략화합니다.

<!--
The actual minimization algorithm is:

  1) Attempt to zero large blocks of data with large stepovers. Empirically,
     this is shown to reduce the number of execs by preempting finer-grained
     efforts later on.

  2) Perform a block deletion pass with decreasing block sizes and stepovers,
     binary-search-style. 

  3) Perform alphabet normalization by counting unique characters and trying
     to bulk-replace each with a zero value.

  4) As a last result, perform byte-by-byte normalization on non-zero bytes.
-->

실제 Minimization 알고리즘은 다음과 같습니다.

  1) Stepover가 있는 큰 데이터 블록의 Zero화를 시도합니다. 경험적으로 이는 미세하게 자원들을 선점하여 실행 횟수를 줄이는 것으로 확인되었습니다.

  2) 블록 크기와 Stepover를 이진 검색 스타일로 줄이는 블록 삭제 Pass를 수행합니다.

  3) Unique한 문자열을 Counting하고, 전체를 Zero화하는 문자 간략화를 수행합니다.

  4) 최종 결과로, 0이 아닌 바이트들에 대해 바이트 단위 간략화를 수행합니다.

<!--
Instead of zeroing with a 0x00 byte, afl-tmin uses the ASCII digit '0'. This
is done because such a modification is much less likely to interfere with
text parsing, so it is more likely to result in successful minimization of
text files.
-->

0x00 Byte로 Zero화하는 대신 afl-tmin은 ASCII 숫자로 '0'을 사용합니다. 이러한 수정이 텍스트 파싱을 방해할 가능성이 훨씬 적기 때문에, 텍스트 파일을 성공적으로 최소화할 가능성이 더 높아집니다.

<!--
The algorithm used here is less involved than some other test case
minimization approaches proposed in academic work, but requires far fewer
executions and tends to produce comparable results in most real-world
applications.
-->

여기에서 사용된 알고리즘은 학술 작업에서 논의된 Test case 최소화 접근법보다 낮은 수준에서 수행되지만, 훨씬 적은 실행이 필요하며 대부분의 실제 응용 프로그램에서 비슷한 결과를 생성합니다.