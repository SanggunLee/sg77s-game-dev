# 동영상에서 애니메이션 만들기

이 문서는 **Unreal Engine 5.8**에서 일반 단일 카메라 동영상을 분석하여 몸 또는 얼굴·몸 애니메이션을 만들고, 조립된 MetaHuman에 적용하는 과정을 설명한다.

{% hint style="warning" %}
UE 5.8의 단일 카메라 몸 애니메이션 처리는 **Windows 전용 Experimental 기능**이다. 제작 환경에서 충분히 테스트한 뒤 사용한다.
{% endhint %}

## 전체 작업 순서

```text
동영상 준비
→ Capture Manager로 가져오기
→ Capture Data 에셋 생성
→ MetaHuman Performance 에셋에서 처리
→ Animation Sequence로 내보내기
→ Level Sequence에서 MetaHuman에 적용
```

## 0. 동영상 준비

처리 결과를 좋게 만들려면 다음 조건을 권장한다.

* 영상에는 **한 명의 연기자만** 등장하게 한다.
* 몸 애니메이션을 추출하려면 머리부터 발끝까지 전신이 항상 보여야 한다.
* 다른 사람, 가구, 소품 등이 연기자의 몸이나 얼굴을 가리지 않게 한다.
* 카메라는 삼각대 등에 고정하고 흔들리지 않게 한다.
* 밝고 고른 조명을 사용하고 모션 블러와 강한 그림자를 피한다.
* 가능하면 HD 또는 4K, 60 FPS 영상을 사용한다.
* 의상과 배경·바닥의 색이 확실히 구분되게 한다.

사람이 겹치거나 신체가 가려진 프레임의 실제 자세는 단일 영상만으로 정확히 복구할 수 없다. 가능하면 해당 구간을 잘라내거나 다른 영상을 사용한다.

공식 참고: [MetaHuman Animator Video Capture Guidelines](https://dev.epicgames.com/documentation/metahuman/metahuman-animator-video-capture-guidelines-in-unreal-engine)

## 1. 필요한 플러그인 준비

1. Epic Games Launcher 또는 Fab에서 **MetaHuman Animator Markerless Motion Capture** 플러그인을 라이브러리에 추가하고 UE 5.8 엔진에 설치한다.
2. Unreal Editor에서 **Edit > Plugins**를 연다.
3. 다음 플러그인을 검색하여 활성화한다.
   * `MetaHuman Animator`
   * `MetaHuman Animator Markerless Motion Capture`
   * `Capture Manager Editor`
4. 에디터 재시작 메시지가 나오면 **Restart Now**를 누른다.
5. 재시작한 뒤 **Tools > Live Link Hub** 메뉴가 표시되는지 확인한다.

몸 동작은 Markerless Motion Capture 플러그인이 필요하다. 얼굴만 처리한다면 MetaHuman Animator의 일반적인 모노 비디오 처리 기능을 사용할 수 있다.

공식 참고: [Getting Started in MetaHuman Animator](https://dev.epicgames.com/documentation/metahuman/getting-started-with-metahuman-animator)

## 2. 적용할 MetaHuman 준비

이미 프로젝트에 조립된 MetaHuman이 있다면 이 단계는 건너뛴다.

1. Content Browser의 빈 공간을 우클릭한다.
2. **MetaHuman > MetaHuman Character**를 선택한다.
3. 생성한 에셋을 열고 Preset에서 캐릭터를 선택한다.
4. **Download Textures**를 누른다.
5. **Create Full Rig**를 누른다.
6. **Assembly**로 이동한다.
7. 캐릭터 이름을 입력하고 **Assemble**을 누른다.
8. 조립이 끝나면 **Save All**을 누른다.

이후 단계에서 말하는 `Visualization Object`와 내보내기 대상은 여기에서 만든 **조립된 MetaHuman**이다. 문서에 적힌 임의의 `BP_캐릭터이름` 경로를 직접 만들 필요는 없다.

## 3. Capture Manager로 동영상 가져오기

### 3.1 Capture Manager 열기

1. Unreal Editor 상단 메뉴에서 **Tools > Live Link Hub**를 선택한다.
2. Live Link Hub 왼쪽 위의 Layout 드롭다운에서 **Capture Manager**를 선택한다.
3. **Data Devices** 패널에서 **Add**를 누른다.
4. **Mono Video Ingest**를 선택한다.

### 3.2 영상 폴더 지정

1. Data Devices 패널에 생성된 Mono Video 장치를 선택한다.
2. 오른쪽 **Details** 패널의 `Source Path`에 영상 파일이 들어 있는 **폴더**를 지정한다.
3. 기본 `Video Discovery Expression`은 `<Auto>`로 둔다.
4. Take Browser에 검색된 영상이 나타나는지 확인한다.

`Source Path`에는 영상 파일 자체가 아니라 영상을 담고 있는 폴더를 지정한다. 하위 폴더도 자동 검색된다.

### 3.3 Ingest 실행

1. Take Browser에서 처리할 영상을 선택한다.
2. **Add to Queue**를 누른다.
3. 선택한 영상이 **Jobs List**에 추가되었는지 확인한다.
4. Pipeline이 `Ingest`인지 확인한다.
5. Jobs List에서 실행 준비가 된 뒤 **Start**를 누른다.
6. 작업이 완료될 때까지 기다린다.

{% hint style="info" %}
`Start`는 Unreal Editor 메인 툴바에 있는 버튼이 아니다. 영상을 **Add to Queue**하여 Jobs List에 작업이 생긴 다음, Capture Manager의 Jobs List에서 누르는 버튼이다.
{% endhint %}

작업이 성공하면 기본적으로 다음 위치에 Capture Data 에셋이 생성된다.

```text
Content/CaptureManager/Imports
```

공식 참고: [Import Your Footage](https://dev.epicgames.com/documentation/metahuman/metahuman-animator-01-import-your-footage-in-unreal-engine)

## 4. MetaHuman Performance 에셋 만들기

Capture Data 에셋 자체에서는 애니메이션을 처리하지 않는다. 별도의 **MetaHuman Performance** 에셋을 만들어야 한다.

1. Content Browser에서 에셋을 저장할 폴더를 연다.
2. 빈 공간을 우클릭한다.
3. **MetaHuman > MetaHuman Performance**를 선택한다.
4. 예를 들어 `MHP_MyVideo`라는 이름을 지정한다.
5. 생성된 MetaHuman Performance 에셋을 더블클릭하여 연다.
6. 상단의 저장 아이콘을 누른다.
7. 이 에셋 편집기의 **Details** 패널에서 `Footage Capture Data`를 찾는다.
8. 앞 단계에서 생성된 Capture Data 에셋을 지정한다.

{% hint style="danger" %}
`Processing Parameters`와 `Visualization Object`는 **Live Link Hub 화면이나 Capture Data 에셋에 있는 항목이 아니다.** Content Browser에서 직접 생성한 **MetaHuman Performance 에셋을 열었을 때 그 편집기의 Details 패널**에 표시된다.
{% endhint %}

## 5. 추출할 애니메이션 설정

MetaHuman Performance 에셋의 Details 패널에서 `Processing Parameters`를 설정한다.

### 몸 애니메이션만 추출

* `Facial Tracking`: Off
* `Body Tracking`: On
* `Head Movement Mode`: Disable

### 얼굴과 몸을 함께 추출

* `Facial Tracking`: On
* `Body Tracking`: On
* `Head Movement Mode`: Disable
* `Processing Parameters > Facial Tracking > Advanced > Models`
  * `Face Detector`: Small Face Detector
  * `Face Solver`: Small Face Solver

전신 영상에서는 얼굴이 화면에서 작게 보이므로 Small 모델을 사용한다. 항목에 Small 모델이 나타나지 않으면 Content Browser의 View Options 또는 Settings에서 **Show Engine Content**를 활성화한 뒤 다시 확인한다.

### 미리보기 캐릭터 지정

1. 같은 MetaHuman Performance 에셋의 Details 패널에서 `Visualization Object`를 찾는다.
2. 에셋 선택기를 열어 앞에서 준비한 **조립된 MetaHuman**을 선택한다.

`Visualization Object`는 결과를 미리 보기 위한 선택 항목이므로 지정하지 않아도 처리는 가능하다. 선택 가능한 에셋 종류는 얼굴만 처리하는지, 몸까지 처리하는지에 따라 달라질 수 있다.

## 6. 영상 처리하기

1. MetaHuman Performance 에셋 하단 타임라인에서 사용할 구간을 정한다.
2. 불필요하거나 사람이 가려진 앞뒤 구간은 범위에서 제외한다.
3. 상단 툴바의 **Process**를 누른다.
4. 처리가 끝날 때까지 기다린다.
5. 재생하여 얼굴, 손발, 몸의 방향과 발 미끄러짐을 확인한다.
6. 범위나 설정을 바꾸었다면 다시 **Process**를 누른다.

## 7. Animation Sequence로 내보내기

1. 결과를 확인한 뒤 상단 툴바의 **Export Animation**을 누른다.
2. 저장할 폴더와 Animation Sequence 이름을 지정한다.
3. MetaHuman에 바로 적용할 것이므로 Skeleton 내보내기 방식으로 **Existing Skeleton**을 선택한다.
4. `Target Skeleton` 또는 `Skeletal Mesh` 항목에서 조립된 MetaHuman의 **Body Skeletal Mesh**를 선택한다.
5. 내보내기를 실행한다.

`Performer Skeleton`은 영상 속 사람의 비율을 기반으로 새로운 골격을 만들고, MetaHuman 이외의 캐릭터로 리타기팅할 때 사용하는 선택지다.

공식 참고: [Process and Export Your Animation](https://dev.epicgames.com/documentation/metahuman/metahuman-animator-02-process-and-export-your-animation-in-unreal-engine)

## 8. Level Sequence에서 MetaHuman에 적용

### 8.1 Level Sequence 준비

1. 사용할 레벨을 연다.
2. Content Browser에서 우클릭하고 **Cinematics > Level Sequence**를 선택한다.
3. Level Sequence 이름을 지정하고 더블클릭하여 연다.
4. 조립된 MetaHuman Blueprint를 Content Browser에서 레벨로 드래그한다.
5. MetaHuman Actor를 선택한다.
6. Sequencer에서 **Add > Add Actor Track**을 누르고 해당 MetaHuman을 선택한다.

MetaHuman이 Sequencer에 추가되면 보통 `Body`와 `Face` 구성 요소가 나타난다.

### 8.2 생성한 애니메이션 연결

1. `Body` 아래에 기본으로 들어 있는 Control Rig 트랙이 있다면 해당 **Control Rig 트랙만** 삭제한다.
2. `Face` 아래의 기본 Control Rig 트랙도 같은 방식으로 삭제한다.
3. `Body` 트랙의 `+` 버튼을 누른다.
4. **Animation**에서 앞 단계에서 내보낸 Animation Sequence를 선택한다.
5. 얼굴과 몸을 함께 처리했다면 `Face` 트랙의 `+`에서도 **같은 Animation Sequence**를 선택한다.
6. Sequencer의 Play 버튼을 눌러 결과를 확인한다.

{% hint style="warning" %}
`Body`와 `Face` 구성 요소 자체를 삭제하는 것이 아니라, 그 아래에 자동 생성된 기존 Control Rig 트랙을 삭제한다. 기존 Control Rig과 Animation 트랙이 동시에 같은 뼈를 제어하면 결과가 충돌할 수 있다.
{% endhint %}

공식 참고: [Apply Your Animation to a MetaHuman](https://dev.epicgames.com/documentation/metahuman/metahuman-animator-03-apply-your-animation-to-a-metahuman-in-unreal-engine)

## 9. 결과 애니메이션 수정하기

간단한 타이밍 수정은 Sequencer에서 애니메이션 클립을 이동하거나 자르고 재생 속도를 바꾸면 된다.

동작 자체를 수정하려면 다음과 같이 Control Rig으로 베이크한다.

* 몸: Body 트랙 우클릭 → **Bake To Control Rig > MetaHuman_ControlRig**
* 얼굴: Face 트랙 우클릭 → **Bake To Control Rig > Face_ControlBoard_CtrlRig**

베이크 후에는 Sequencer와 뷰포트에서 손, 발, 골반, 표정 컨트롤의 키프레임을 직접 수정할 수 있다. 원본 Animation Sequence는 복제하여 보관하는 것이 안전하다.

## 문제 해결

### Start 버튼이 보이지 않는다

* Live Link Hub의 Layout이 `Capture Manager`인지 확인한다.
* Take Browser에서 영상을 선택하고 `Add to Queue`를 먼저 누른다.
* 영상이 Jobs List에 들어갔는지 확인한다.
* Pipeline이 `Ingest`인지 확인한다.

### Processing Parameters가 보이지 않는다

현재 열어 둔 에셋을 확인한다. `Capture Data`가 아니라 직접 생성한 **MetaHuman Performance** 에셋을 열어야 한다.

### Body Tracking이 보이지 않는다

* Windows에서 UE 5.8을 사용하고 있는지 확인한다.
* `MetaHuman Animator Markerless Motion Capture`가 엔진에 설치되고 프로젝트에서 활성화되었는지 확인한다.
* 플러그인 활성화 후 에디터를 재시작했는지 확인한다.

### Visualization Object를 선택할 수 없다

* MetaHuman Character에 Full Rig을 생성했는지 확인한다.
* Assembly 단계에서 조립을 완료하고 Save All을 실행했는지 확인한다.
* 임의의 Blueprint 경로를 만들지 말고 에셋 선택기에 실제로 표시되는 조립된 MetaHuman을 선택한다.

### 처리 결과가 심하게 흔들리거나 다른 사람을 추적한다

MetaHuman Animator의 단일 카메라 몸 추출은 한 명의 연기자를 전제로 한다. 사람이 겹치는 구간, 신체가 프레임 밖으로 나가는 구간, 심한 모션 블러가 있는 구간을 제거하고 다시 처리한다.

## 공식 문서

* [Animation from Mono Video Capture](https://dev.epicgames.com/documentation/metahuman/metahuman-animation-from-mono-video-capture-in-unreal-engine)
* [Import Your Footage](https://dev.epicgames.com/documentation/metahuman/metahuman-animator-01-import-your-footage-in-unreal-engine)
* [Process and Export Your Animation](https://dev.epicgames.com/documentation/metahuman/metahuman-animator-02-process-and-export-your-animation-in-unreal-engine)
* [Apply Your Animation to a MetaHuman](https://dev.epicgames.com/documentation/metahuman/metahuman-animator-03-apply-your-animation-to-a-metahuman-in-unreal-engine)
* [Video Capture Guidelines](https://dev.epicgames.com/documentation/metahuman/metahuman-animator-video-capture-guidelines-in-unreal-engine)

