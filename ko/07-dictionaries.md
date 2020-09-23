### 7) Dictionaries
---
<!-- 
The feedback provided by the instrumentation makes it easy to automatically identify syntax tokens in some types of input files, and to detect that certain combinations of predefined or auto-detected dictionary terms constitute a valid grammar for the tested parser. 
-->

Instrumentation에 의해서 제공된 피드백은 몇가지 유형의 인풋 파일들안에 있는 문법 Token들을 자동으로 식별하는 것을 쉽게 만들어 줍니다. 그리고 또, 먼저 정의되거나 자동으로 감지된 딕셔너리 용어의 특정한 조합이 테스트된 파서에 유효한 문법을 구성하는지 여부를 감지하는 것도 쉽게 만들어 줍니다.

<!-- 
A discussion of how these features are implemented within afl-fuzz can be found here:

  http://lcamtuf.blogspot.com/2015/01/afl-fuzz-making-up-grammar-with.html
-->

이 특징들이 어떻게 afl-fuzz에 구현되었는지 궁금하면 여기를 방문해보세요.

  http://lcamtuf.blogspot.com/2015/01/afl-fuzz-making-up-grammar-with.html

<!-- 
In essence, when basic, typically easily-obtained syntax tokens are combined together in a purely random manner, the instrumentation and the evolutionary design of the queue together provide a feedback mechanism to differentiate between meaningless mutations and ones that trigger new behaviors in the instrumented code - and to incrementally build more complex syntax on top of
this discovery.
 -->

본질적으로, 쉽게 얻은 기본적인 문법 토큰들이 순전히 랜덤한 방식으로 결합될때, Instrumentation과 큐의 진화된 디자인은 Instrumented code안에서의 새로운 행동을 트리거하는 것들과 의미없는 뮤테이션들을 구분하고 이 발견들로 말미암아 점진적으로 더 복잡한 문법들을 작성하기 위한 위한 피드백 매커니즘을 제공한다.

<!-- 
The dictionaries have been shown to enable the fuzzer to rapidly reconstruct the grammar of highly verbose and complex languages such as JavaScript, SQL, or XML; several examples of generated SQL statements are given in the blog post mentioned above.
 -->

딕셔너리들은 퍼저가 장황하고 복잡한 JavaScript, SQL, XML같은 언어의 문법을 빠르게 재설계 할 수 있도록 합니다. 생성된 SQL문의 예시는 위에서 언급한 블로그 포스트에서 확인해 볼 수 있습니다.

<!-- 
Interestingly, the AFL instrumentation also allows the fuzzer to automatically isolate syntax tokens already present in an input file. It can do so by looking for run of bytes that, when flipped, produce a consistent change to the program's execution path; this is suggestive of an underlying atomic comparison to a predefined value baked into the code. The fuzzer relies on this signal to build compact "auto dictionaries" that are then used in conjunction with other fuzzing strategies.
 -->

재밌게도, AFL Instrumentation은 퍼저가 자동으로 이미 인풋 파일안에 존재하는 문법 토큰을 분리할 수 있게 만들어 줍니다. 어떻게 그게 가능하냐면 Flipped될때, 프로그램의 실행경로를 일관되게 변경하는 바이트들의 실행을 찾아서 그렇게 할 수 있습니다. 이말인 즉슨, Underlying Atomic와 코드안에서 baked된 미리 정의된 값과의 비교를 암시합니다. 퍼저는 다른 퍼징 전략들과 더불어 사용될 수 있는 컴팩트한 "자동 Dictionaries"들을 만들기 위해서 이 신호에 의존합니다.