### 2) Detecting new behaviors
---
The fuzzer maintains a global map of tuples seen in previous executions; this data can be rapidly compared with individual traces and updated in just a couple of dword- or qword-wide instructions and a simple loop.

튜블의 글로벌 맵(데이터)은 individual traces들이랑 빠르게 비교되기 + dword나 qword-wide instruction들과 간단한 loop로 업데이트될 수 있다.

mutated input은 execution trace(글로벌 맵에 들어가는 튜플을 포함한, 이 튜플들로 갈색을 할 수 있다.)를 만들고 인풋파일은 보존되고, 나중에 추가적인 실행을 위해 쓰인다.

커버리지를 안 넓히는 애들은 다 버린다. 아무리 control flow가 고유해도. 고유한 거랑 local-scale state transitions을 찾는거는 다른듯

state transitions 못찾는애들 다 가져다 버리는 방법! 요거 쓰면 귀찮은 비교랑 경로 터지는 문제없이 세밀하고 긴 탐구가 가능하다.

한번 알고리즘의 속성을 이해해봅시다. 아래에 두번째 trace는 새로운 튜플인 CA, AE가 있으니까 사실상 새로운 것일 가능성이 크다.

#2가 실행될때, 같이 #3를 살펴보면 요 패턴이 뭔가 고유한거 같지는 않아도, 분명히 전체적으로 다른 execution path를 가지고 있는것을 확인할 수 있다. 전체 실행경로가 딱봐도 달라도, unique하다고 판단하지 않는다. 그니까 유니크 한게 중요한게 아니라 전체 실행경로가 다른것만 봐야한다.
(unique하다고 해서 중요한게 아님!!!)

새로운 튜플들을 탐지하는거 말고도, 퍼저는 항상 튜플 적중 횟수를 고려해야한다. 요것들은 여러 Buckets로 나뉜다.(여러 덩어리로 묶이며 나뉜다)

어느 정도, 그 Buckets들의 수는 Implementation Artifact 이다. 이를 이용해서 8비트 카운터의 In-place mapping이 가능합니다. 여기서 8비트 카운터는 8-position Bitmap에 의한 Instrumentation에 의해서 생성됩니다. 또한, 8-position Bitmap은 각 튜플에 대하여 이미 확인한 실행 횟수들을 지속적으로 추적(track)을 하기 위해 퍼저에 의존합니다.

어느 정도, 그 Buckets들의 수는 Implementation Artifact 이다. 이를 이용해서 8비트 카운터의 In-place mapping이 가능합니다. 여기서 8비트 카운터는 8-position Bitmap에 의한 Instrumentation에 의해서 생성됩니다. 또한, 8-position Bitmap은 각 튜플에 대하여 이미 확인한 실행 횟수들을 지속적으로 추적(track)을 하기 위해 퍼저에 의존합니다.

Hit count behavior는 잠재적으로 흥미로운 컨트롤 플로우 변화를 구분하는 방법을 알려준다. 예를 들어서 코드의 블록을 딱 한번 hit했는데 두번 실행되는 경우는 흥미로운 컨트롤 플로우 변화에 해당한다.

동시에, 경험적으로 덜 중요해보이는 변화에는 상당히 둔감하게 반응한다. 예를들어 루프가 47cycles에서 48로 변화하는것은 관심이 없다. 또, 그 카운터들은 조밀한 trace maps에서 튜플 충돌에 대해 어느정도 "우연한" 면역을 준다. (카운터들은 가끔가다 우연히 튜플 충돌 터져도 문제 없이 지켜줄때가 있다.)

실행은 메모리와 실행 시간 제한에 의해서 상당히 엄격하게 감시됩니다.  기본적으로 타임아웃은 초기 보정된 실행 속도의 5배인 약 20m/s로 설정됩니다. 공격적인 타임아웃은 극적인(Dramatic) 퍼저 성능 저하를 방지하는 것을 의미합니다. 극적인 퍼저 성능저하는 Tarpits로의 하강에 의해 발생하는데, 이게 뭔 뜻이냐면 coverage를 1%개선하는데 100배 느려지는 것을 의미합니다. 우리는 실용적으로 이 방법들을 거부하고 퍼저가 적은 비용으로도 같은 코드에 도달할 수 있는 방법을 찾기를 바랍니다. 그동안 진행했던 테스트들은 더 많은 너무 많이 시간제한을 두는 것은 투자대비 가치가 없다는 것을 알려주고 있습니다. (소용이 없다는 것을 강력하게 알려주고 있습니다.)