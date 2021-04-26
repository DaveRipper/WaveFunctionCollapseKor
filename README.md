# WaveFunctionCollapseKor
[원본](https://github.com/mxgmn/WaveFunctionCollapse)

이 프로그램은 입력된 비트맵과 지역적으로 유사한 비트맵들을 생성합니다.

![](https://raw.githubusercontent.com/mxgmn/Blog/master/resources/wfc.png)
![](https://camo.githubusercontent.com/6156e7afcb795d6a05fe7f02a43b5a477ad6eec8df3293db7509b4d6aa7d830f/687474703a2f2f692e696d6775722e636f6d2f734e75425653722e676966)

지역적으로 유사하다는 것은 다음을 의미합니다.
* (조건 1) 생성된 결과물의 각 NxN 픽셀 패턴이 입력에서 적어도 한 번 나타남.
* (약조건 2) 입력에서의 NxN 패턴 분포는 충분히 큰 개수의 출력에서의 NxN 패턴 분포와 유사해야 함. 다시 말해, 출력에서 특정한 패턴이 생성될 가능성은 입력에서 해당 패턴의 밀도에 근접해야 함.

아래 예시에서의 N 크기는 3입니다.

![](https://raw.githubusercontent.com/mxgmn/Blog/master/resources/wfc-patterns.png)

WFC는 완전한 비관찰 상태(unobserved state)에서 비트맵을 초기화합니다. 비관찰 상태란 각각의 픽셀값이 입력 비트맵의 색상들이 중첩된 상태를 말합니다. (즉 입력이 흑백이라면 비관찰 상태는 회색 음영으로 표시됩니다) 이러한 중첩 계수는 복소수가 아닌 실수이므로 양자역학이 성립하지 않지만 그 원리에서 영감을 받았습니다. 이후 프로그램은 관찰-전파 주기(observation-propagation cycle)에 돌입합니다.
* 각 관찰 단계마다, NxN 구역은 가장 낮은 섀넌 앤트로피를 보이는 비관찰 상태 중에서 선택됩니다. 이 구역의 상태는 계수와 입력의 NxN 패턴 분포에 따른 유한한 상태로 붕괴됩니다.
* 각 전파 단계마다, 이전 단계의 붕괴로 얻어진 새로운 정보가 출력을 통해 전파됩니다.

각각의 단계마다 전체 엔트로피는 감소하며 결국 완전히 관찰된 상태를 획득하게 되며, 파동 함수가 붕괴됩니다.

전파 단계를 진행하는 도중에 특정한 픽셀에 대한 모든 계수가 0이 될 수 있습니다. 이것은 알고리즘이 모순을 맞닥뜨려 더 이상 진행할 수 없다는 의미입니다. 특정한 비트맵이 조건 1을 충족하는 다른 비자명 비트맵을 허용하는지 확인하는 문제는 NP-난해이므로, 항상 완료되는 빠른 해결책을 만드는 건 불가능합니다. 그러나 실제로, 이러한 상황은 대단히 드물게 발생합니다.


## 알고리즘
1. 입력 비트맵을 읽어들이고 NxN 패턴의 개수를 셉니다.
    1. (선택 사항) 회전 및 반전을 통해 패턴 데이터의 양을 증가시킵니다.
2. 출력의 차원에 관한 배열을 만듭니다. (소스에서는 "파동"이라고 부릅니다) 이 배열의 각 요소는 출력의 NxN 구역의 상태를 나타냅니다. NxN 구역의 상태는 부울 계수로 나타낸 입력의 NxN 패턴 중첩을 말합니다. (즉 출력에서의 픽셀의 상태는 입력된 색상들의 실제 계수의 중첩입니다.) 거짓(False) 계수는 해당하는 패턴이 금지되었다는 것을 의미하며, 참 계수는 해당하는 패턴이 아직 금지되지 않았다는 것을 의미합니다.
3. 파동을 완전한 비관찰 상태로 초기화합니다. 즉 모든 부울 계수를 참으로 만듭니다.
4. 다음 단계들을 반복합니다.
    1. 관찰:
        1. 0이 아닌 최소의 엔트로피를 갖는 파동 요소를 찾습니다. 이러한 요소가 없다면 (모든 요소가 0 또는 정의되지 않은 엔트로피를 갖는다면) 4단계를 끝내고 5단계로 넘어갑니다.
        2. 해당 요소를 자신의 계수와 입력의 NxN 패턴 분포에 따른 유한한 상태로 붕괴시킵니다.
    2. 전파: 이전 관찰 단계를 통해 전파 정보를 획득합니다.
5. 이쯤이면 모든 파동 요소가 완전히 관찰된 상태거나(하나를 제외한 모든 계수가 0이 됨) 모순적인 상태(모든 계수가 0)일 것입니다. 전자의 경우 출력을 반환합니다. 후자의 경우 아무것도 반환하지 않고 알고리즘을 종료합니다.


## 타일맵 생성
이 알고리즘의 가장 단순한 비자명한 예시는 NxN이 1x2일 때입니다. (사실 NxM이죠.) 만약 색상쌍(pairs of colors)의 확률을 저장하는 게 아닌, 색상 자체의 확률을 저장함으로써 해당 예시를 더욱 간략화한다면, 우리는 "단순 타일 모델(simple tiled model)"이라 불리는 것을 얻게 됩니다. 이 모델에서의 전파 단계는 그저 인접 제한 전파(adjacency constraint propagation)입니다. 단순 타일 모델을 샘플 비트맵이 아닌 타일 목록과 해당 타일들의 인접 데이터(인접 데이터는 매우 작은 샘플의 큰 집합으로 볼 수 있습니다)를 가지고서 초기화하는 것이 편리합니다.

[GIF](http://i.imgur.com/jIctSoT.gif)

실제 타일셋에서 가능한 모든 인접 타일쌍의 목록은 상당히 길 수 있으므로, 저는 열거 과정(enumeration)을 단축하기 위해 타일 대칭 시스템을 구현했습니다. 이 시스템에서 각 타일은 대칭 유형과(symmetry type) 함께 할당되어야 합니다.

![](https://raw.githubusercontent.com/mxgmn/Blog/master/resources/wfc-symmetry-system.png)

타일은 할당된 문자와 동일한 대칭 유형을 갖습니다. (즉 정이면체군 D4의 동작은 타일 및 해당 문자에 대해 동형입니다.) 이 시스템을 사용하면 인접 타일쌍을 대칭까지 열거하며, 대칭 타일이 많은 타일셋의 인접 목록을 몇 배는 더 짧게 만들 수 있습니다.

![](https://raw.githubusercontent.com/mxgmn/Blog/master/resources/wfc-knots.png)   
매듭 타일셋
   
![](https://raw.githubusercontent.com/mxgmn/Blog/master/resources/wfc-rooms.png)   
방 타일셋
   
![](https://raw.githubusercontent.com/mxgmn/Blog/master/resources/wfc-circuit-1.png)
![](https://raw.githubusercontent.com/mxgmn/Blog/master/resources/wfc-circuit-2.png)   
회로 타일셋
   
![](https://raw.githubusercontent.com/mxgmn/Blog/master/resources/wfc-circles.png)   
원 타일셋
   
![](https://raw.githubusercontent.com/mxgmn/Blog/master/resources/wfc-castle.png)   
성 타일셋
   
![](https://raw.githubusercontent.com/mxgmn/Blog/master/resources/wfc-summer-1.png)
![](https://raw.githubusercontent.com/mxgmn/Blog/master/resources/wfc-summer-2.png)   
여름 타일셋

제한되지 않는 매듭 타일셋(타일 5개가 모두 허용됐을 때)은 타일을 배치할 수 없는 상황이 생기지 않기 때문에 WFC로 다루기에는 흥미롭지 않습니다. 우리는 이러한 속성을 갖는 타일셋을 "수월하다"고 부릅니다. 특별한 휴리스틱 없이는 수월한 타일셋들로 흥미로운 전역적 배열을 생성해낼 수 없습니다. 수월한 타일셋의 타일 상관관계는 거리에 따라 빠르게 감소하기 때문입니다. 더 많은 수월한 타일셋은 [cr31의 사이트](http://cr31.co.uk/stagecast/wang/tiles_e.html)에서 찾아볼 수 있습니다. 해당 사이트에서 "Dual" 타일셋을 살펴보세요. 이 수월한 타일셋은 어떻게 매듭을 생성할까요? 정답은 좁은 종류의 매듭만 생성해내고, 임의의 매듭은 생성해낼 수 없습니다.

또한 회로, 여름, 방 타일셋은 [왕 타일](https://en.wikipedia.org/wiki/Wang_tile)이 아닙니다. 즉, 해당 타일셋들의 인접 데이터는 가장자리 색상에서 유도될 수 없습니다. 예를 들어, 회로 타일셋에서 두 모서리는 인접할 수 없지만 연결 타일을 통해 연결될 수 있으며, 대각선 경로는 방향을 바꿀 수 없습니다.
