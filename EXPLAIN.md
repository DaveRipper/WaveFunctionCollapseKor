[원본](https://twitter.com/exppad/status/1267045322116734977)

@ExUtumno가 텍스쳐 합성의 한 가지 방법으로서 소개한 #WaveFunctionCollapse 알고리즘은 @OskSta에 의해 3D 빌딩 생성에 꽤 멋지게 쓰이고 있습니다. 잠시만요, 뭐라구요? 어떻게 두 개가 연관이 있을 수가 있죠? 완전히 다른 분야인 것 같은데요!   
![](https://pbs.twimg.com/media/EZVxd-DWAAE5cNQ?format=jpg&name=large)
*같은 알고리즘?*
   
별개의 픽셀값: WFC는 여타 텍스쳐 합성 알고리즘과는 다릅니다. 픽셀 색상을 절대로 혼합하지 않기 때문이죠. WFC는 약간의 파랑색과 붉은색이 만나면 보라색이 된다는 걸 알 필요가 없습니다. 다른 알고리즘은 알아야 하는데도 말이죠.
![](https://pbs.twimg.com/media/EZVxhS3XYAAzT-E?format=jpg&name=large)

이 점은 WFC를 좀 더 범용적이게 만드는 강점으로서 작용합니다. 픽셀은 색상을 포함할 필요가 없습니다. 그저 뭔가 "물건", 더 정확히 하자면, "추상적인 심볼"을 포함하기만 하면 됩니다. 추상적 심볼은 정수, 문자, 도형 등 무엇이든지 될 수 있습니다.
![](https://pbs.twimg.com/media/EZVxpkDX0AAt_NW?format=jpg&name=large)

WFC에는 몇 가지 단점이 있기는 합니다. 왜 기존 예시들이 전부 픽셀아트 느낌인지 궁금해하지 마세요! 이 알고리즘은 한 픽셀의 색상이 다른 색상과 "유사"하다는 것을 인식하지 못합니다. WFC에게는 두 색이 굉장히 비슷하거나 완전히 다르게 보입니다.
![](https://pbs.twimg.com/media/EZVxtDiXQAAYliD?format=jpg&name=large)
*안티 앨리어싱된 선처럼 보이시나요? 색상들은 모두 동일하게 동떨어져 있습니다. 이게 WFC가 선을 바라보는 모습입니다!*
   
그래서, WFC가 정확히 어떤 문제를 해결한다는 걸까요? 두 가지 주요한 단계가 있습니다. 첫 번째는 예시 이미지로부터 타일과 인접 규칙(adjacency rules)을 추출(extraction) (혹은 "학습(learning)"이라고도 할 수 있지만, 이렇게 부르는 건 너무 과한 것 같습니다) 하는 것입니다.
![](https://pbs.twimg.com/media/EZVxwfAXkAELKCY?format=png&name=large)
*예시 ---추출--> "타일" 혹은 "패턴" / 인접 규칙 (오른쪽에 배치될 수 있음, 왼쪽에 배치될 수 있음, 아래에 배치될 수 있음 ...)*
   
두 번째 단계는 제약 해결 (constraint solving) 입니다. 새 이미지의 각 위치 - 혹은 "슬롯" - 마다, 우리는 근처 타일들의 인접 규칙을 지키는 타일 하나를 고릅니다. @OskSta가 이 단계를 보여주는 [멋진 데모](https://t.co/x2OrFyVY9N?amp=1)를 만들었습니다.
![](https://pbs.twimg.com/media/EZVx0UBXgAAv4qi?format=jpg&name=large)
*인접 규칙의 목록 안에 있어야 합니다*
   
원본 구현에서 두 가지 파생형을 찾아볼 수 있습니다: "타일(tiled) 모델"과 "중복(overlapping) 모델"입니다. 전자는 입력 이미지를 단순히 NxN 패턴으로 자르고 (여기서는 3x3) 이 패턴들을 "추상적 심볼"로서 사용합니다.
![](https://pbs.twimg.com/media/EZVx35nWkAQapCu?format=png&name=large)
*타일 모델에서, 왼쪽의 두 가지 문제는 동일하게 다뤄집니다.*
   
중복 모델은 일종의 미닫이창을 이용해 패턴을 추출합니다. 하나의 타일은 이웃한 픽셀의 정보를 함께 포함하고 있는 하나의 픽셀이 됩니다. 인접 규칙은 이러한 이웃으로부터 얻어집니다.
![](https://pbs.twimg.com/media/EZVx7vGXQAA7gyA?format=jpg&name=large)
*예시 / 미닫이창 / 추출된 타일*   
*오른쪽에 파란색, 왼쪽에 녹색 이웃을 가진 파란색 픽셀*
   
슬롯의 개념은 모델에 따라 달라집니다. 그러나 어쨌듯 출력 이미지를 완성하기 위해서는 여러 개의 슬롯들을 추상적 심볼로 채워야 합니다.
![](https://pbs.twimg.com/media/EZVx_hyXkAE3K5p?format=jpg&name=large)
*이 캔버스는:   
- 타일 모델에서는 한 개의 슬롯이 됩니다.
- 중복 모델에서는 9개의 슬롯이 됩니다.*
   
타일 모델과 중복 모델의 진정한 차이점은 슬롯 사이의 관계입니다. 중복 모델에서 관계의 수가 더욱 많습니다. 패턴이 3x3보다 크다면, 인접 유형이 이미지보다 훨씬 많아질 것입니다.
![](https://pbs.twimg.com/media/EZVyCfAX0AA84zA?format=png&name=large)
*사각형: 슬롯, 선: 인접*
   
그러나 대체로 두 개의 WFC 모델은 결국 같습니다. 출력 이미지를 채우는 단계는 어떤 모델인지에 따라 달라지지 않습니다. 결국 전부 타일, 슬롯, 인접 및 규칙에 관한 작업일 뿐이니까요.

그렇다면 3D는 어떨까요? 다시 한 번 인접에서 변화가 일어납니다! 타일 모델에서는 왼쪽, 오른쪽, 앞, 뒤, 위, 아래 총 6개의 인접 관계가 있습니다. 중복 모델도 같습니다.
![](https://pbs.twimg.com/media/EZVyHiiWAAAF0At?format=jpg&name=large)
*선: 인접, 점: 슬롯, 3D 모델들: 타일*
   
좋습니다, 그럼 출력물을 생성하는 건 어떻게 되나요? 격자가 아니라면요? "아무 상관 없습니다." 다음은 구체에서의 예시입니다. [#1](https://twitter.com/boris_brave/status/1244387278732091392) [#2](https://t.co/L9iQrzglFW?amp=1)
![](https://pbs.twimg.com/media/EZVyLb1WsAA2kEl?format=jpg&name=large)
*선: 인접, 핑크색 조각: 슬롯*
   
WFC는 불규칙한 격자에서조차 작동합니다. 다음은 [#Townscape](https://store.steampowered.com/app/1291340/Townscaper/)를 제가 구현해본 모습입니다. 눈에 띄는 변경점은 이제 인접 관계가 대칭일 필요가 없다는 점입니다. 만약 슬롯 A가 슬롯 B의 "앞에" 있다면, B는 A "뒤에" 있을 필요가 없습니다.
![](https://pbs.twimg.com/media/EZVyUUQXkAEiu46?format=jpg&name=large)
*핑크색 점: 슬롯, 파란색 화살표: 인접 (그림에서는 예시로 "앞에 있음"을 나타냄)*
   
이제 다 왔습니다. Gumin의 텍스쳐 합성과 Stalberg의 절차적 마을 생성은 동일한 파동 함수 붕괴 알고리즘을 사용합니다. 그저 타일, 인접 규칙과 슬롯의 위상에 변화를 주었을 뿐입니다!
![](https://pbs.twimg.com/media/EZVybLqX0AE_RCv?format=jpg&name=large)
* 타일 / 인접 규칙 / 슬롯 위상*
   
픽셀이 색상을 포함하지 않습니다. 이미지가 더 이상 격자 형태가 아닙니다. 우리는 텍스쳐 합성의 접근 방식을 완전히 부숴버렸습니다. 그러나 다시 말씀드리지만, 이건 좋은 소식입니다! 해당 사항들은 파동 함수 붕괴 알고리즘의 다양성, 융통성을 보여줍니다.

주의: 저는 @OskSta가 정확히 어떻게 연결 규칙을 구현했는지 잘 모르겠습니다. 추측만 할 뿐이죠. 다음은 제가 어떻게 구현했는가입니다: 타일의 면이 어떠한 타일과 연결될 수 있는지 나타내도록 분류했습니다.
![](https://pbs.twimg.com/media/EZVyjHEXkAM5AT8?format=jpg&name=large)
