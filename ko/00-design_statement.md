### 0) Design statement
---

<!--
American Fuzzy Lop does its best not to focus on any singular principle of
operation and not be a proof-of-concept for any specific theory. The tool can
be thought of as a collection of hacks that have been tested in practice,
found to be surprisingly effective, and have been implemented in the simplest,
most robust way I could think of at the time.
-->

American Fuzzy Lop은 단일한 원칙에 초점을 맞추거나 특정 이론에 대한 개념 증명(Proof-of-Concept)가 되지 않으려 노력합니다. AFL은 실제로 테스트를 거쳐 효과성이 입증된 기술들의 모음으로 간주됩니다. 동시에 놀랍도록 효과적이지만 간단하게 구현되었습니다.

<!--
Many of the resulting features are made possible thanks to the availability of
lightweight instrumentation that served as a foundation for the tool, but this
mechanism should be thought of merely as a means to an end. The only true
governing principles are speed, reliability, and ease of use.
-->

아래와 같은 원리로 인해 빠른 instrumentation이 가능하였고 AFL의 많은 기능들이 동작할 수 있었지만, 이는 단순히 fuzzing을 위한 수단으로 여겨져야 합니다. 지배적인 원칙은 속도, 신뢰성, 사용하기 쉬움, 사용의 용이성 뿐입니다.