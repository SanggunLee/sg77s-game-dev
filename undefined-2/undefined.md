# - 카메라



{% embed url="https://www.youtube.com/watch?v=mVFKUOqG3CI" %}

발더스3 와 같은 방식으로 한다.

1. Top Down
2. 카메라가 케릭터와 떨어져서 이동
3. 카메라가 지면과 가까워지면 수평으로 각도 조절
4. Q,E 로 현 지점을 중심으로 회전
5. 케릭터 초상화 더블클릭 시 케릭터에게 초점



<details>

<summary>GPT-o1 대답</summary>

아래는 언리얼 엔진의 기본 **Top Down 템플릿**을 이용하되, “**발더스 게이트 3**”처럼 **카메라가 캐릭터와 완전히 분리**되어 자유롭게 움직이고, 상황에 따라 **각도**를 바꾸고, **Q/E로 회전**하며, **캐릭터 초점**으로 빠르게 이동할 수 있게 만드는 구현 방안입니다.

***

## 전체 개요

1. **Top Down 템플릿** 시작점
   * 기본 템플릿에는 플레이어 캐릭터를 위한 **Pawn/Character** + Camera(SpringArm)가 있습니다. 보통은 캐릭터 위에서 카메라가 따라가는 구조지만, 이 예제에서는 **카메라를 독립**하여 캐릭터와 분리시키는 방법을 사용할 것입니다.
   * 즉, **캐릭터**는 “게임 내 아바타”일 뿐이고, **카메라 이동/회전**은 별도의 “카메라 Pawn(또는 SpectatorPawn)”이 담당합니다.
2. **Camera Pawn(또는 SpectatorPawn)** 생성
   * 플레이어가 조작(이동, 회전, 줌 등)하는 것은 캐릭터가 아니라 “카메라 전용 Pawn” 입니다.
   * **카메라 Pawn** 안에 **SpringArm + CameraComponent**를 구성합니다.
3. **카메라 이동/회전/줌 인/줌 아웃** 등 컨트롤 로직
   * W, A, S, D(또는 마우스 화면 테두리)로 평면 이동
   * 마우스 휠로 줌 인/아웃 (SpringArm->TargetArmLength)
   * 카메라가 땅과 가까워지면 Pitch(각도) 자동 조정 (Distance 측정 후 보간)
   * Q/E로 “현재 카메라 위치를 기준”으로 Yaw 회전
   * 캐릭터 초상화 더블 클릭 시, 그 캐릭터 위치로 **초점 이동** (카메라 위치/회전 보간 이동)

***

## 순서별 상세 구현

아래는 **중요 순서**대로 설명합니다. (언리얼 에디터 기준, 블루프린트로 설명)

### 1. TopDown 템플릿 프로젝트 생성

1. 언리얼 엔진 실행 → **새 프로젝트** → **Blueprint** 탭 → **Top Down** 템플릿 선택
2. 프로젝트 생성 후 실행하면, 기본적으로 **TopDownCharacter**가 맵에 배치되고, 카메라가 캐릭터 위쪽에서 내려다보는 구도로 플레이가 시작됩니다.

**주의:** 이 템플릿은 “캐릭터에 SpringArm + CameraComponent”가 붙은 구조로 되어 있어, 마우스 클릭으로 캐릭터가 이동하는 형태입니다. 우리는 이것을 수정하거나 대체할 것입니다.

***

### 2. Camera Pawn(또는 SpectatorPawn) 생성

1. **콘텐츠 브라우저** → **우클릭** → **Blueprint Class** 생성
   * 부모 클래스로 **Spectator Pawn**(또는 Pawn)을 선택
   * 예: **BP\_FreeCamPawn** 라고 명명
2. **BP\_FreeCamPawn** 열기 → **Components 패널**에 컴포넌트 추가
   1. `DefaultSceneRoot`(혹은 `SceneComponent`)가 루트
   2. **SpringArm**(이름 예: “CameraSpringArm”) 추가
      * `TargetArmLength` 기본값: 예) 1200.0
      * `bDoCollisionTest = false` (필요에 따라 켜거나 끌 수 있음)
      * `bInheritPitch`, `bInheritYaw`, `bInheritRoll` 설정(필요 시)
   3. **Camera**(이름 예: “FollowCamera”) 추가하고, SpringArm의 소켓에 연결
3. **Class Defaults**(또는 **Details**)에서 `Auto Possess Player`를 **Player0**(또는 “Disabled” 후에 직접 Possess)로 설정해도 되고, 실행 시 PlayerController가 이 Pawn을 Possess하도록 해도 됩니다.

***

### 3. 게임 시작 시 카메라 Pawn이 사용되도록 세팅

1. **World Settings**(또는 Project Settings) → “GameMode Override”가 **TopDownGameMode**일 것입니다.
2. **TopDownGameMode**(또는 새로운 GameMode 생성)에서 **Default Pawn Class**를 `BP_FreeCamPawn` 으로 변경
   * 이렇게 하면 플레이 시작 시 자동으로 `BP_FreeCamPawn`이 생성되어 PlayerController가 그것을 Possess합니다.

혹은, Blueprint로 직접 **BeginPlay** 때 `PlayerController->Possess(BP_FreeCamPawn)` 또는 `SetViewTargetWithBlend()`를 호출하는 방식도 가능합니다.

***

### 4. 기존 TopDownCharacter의 카메라 로직 비활성화/삭제

* 기본 템플릿은 **TopDownCharacter** 안에 SpringArm + Camera가 붙어 있습니다.
* 또한 **TopDownPlayerController**가 “마우스 클릭하면 해당 위치로 캐릭터가 이동” 하는 로직을 가지고 있습니다.
* 이번에 우리는 “캐릭터 움직이는 로직”만 남기고, “카메라 조작”은 **FreeCamPawn**에서 처리할 것이므로, **TopDownCharacter**의 CameraComponent를 사용하지 않도록 합니다.
  * **TopDownCharacter** 블루프린트 열기 → Components 패널에서 Camera와 SpringArm을 **삭제**하거나, 설정에서 **비활성화**(Visible/Active 체크 해제)합니다.
  * PlayerController의 “SetViewTarget”이 TopDownCharacter로 되어 있는 부분이 있으면 제거하거나 주석 처리합니다.

***

### 5. FreeCamPawn에 카메라 이동 및 회전 입력 바인딩

이제 **BP\_FreeCamPawn**에서 캐릭터 대신 카메라를 이동/회전/줌 하는 로직을 구현합니다. 대표적인 로직:

1. **Input Axis** 바인딩 (Project Settings → Input)
   * 예: `MoveForward` (W, S 키) → 축 값(+1, -1)
   * 예: `MoveRight` (A, D 키) → 축 값(+1, -1)
   * 예: `CameraZoom` (Mouse Wheel) → 축 값(+1, -1)
2. **Input Action** 바인딩
   * 예: `RotateLeft` (Q 키)
   * 예: `RotateRight` (E 키)

#### 5.1. 평면 이동 (W, A, S, D)

*   **BP\_FreeCamPawn**의 Event Graph에서:

    ```plaintext
    InputAxis MoveForward (Axis value: Val)
        -> AddMovementInput(CameraForwardVector, Val)

    InputAxis MoveRight (Axis value: Val)
        -> AddMovementInput(CameraRightVector, Val)
    ```

    * `CameraForwardVector`와 `CameraRightVector`는 월드 공간 기준으로 XY 평면에서 움직일 수 있게 계산합니다.
    * 예: `GetActorForwardVector()`에서 Z를 무시하거나, SpringArm의 Yaw 회전만 가져와서 평면 방향 벡터로 사용.

#### 5.2. 줌 인/줌 아웃 (마우스 휠)

*   SpringArm의 `TargetArmLength` 값을 변경:

    ```plaintext
    InputAxis CameraZoom (Axis value: ZoomVal)
        -> SpringArm->TargetArmLength = Clamp(CurrentLength - ZoomVal * ZoomSpeed, MinZoom, MaxZoom)
    ```

    * 마우스 휠을 굴리면 +1, -1 값이 들어오므로, 이를 기반으로 길이를 늘리거나 줄입니다.

#### 5.3. Q/E 회전

*   Yaw 회전 제어:

    ```plaintext
    InputAction RotateLeft (Pressed)
        -> AddActorLocalRotation(Rotator(0, -RotationSpeed, 0))

    InputAction RotateRight (Pressed)
        -> AddActorLocalRotation(Rotator(0, +RotationSpeed, 0))
    ```

    * **회전 중심**을 “Pawn의 현재 위치”로 삼습니다.
    * 회전을 누르고 있는 동안 매 프레임 조금씩 회전시키려면, Input Axis처럼 구현하거나, 혹은 단발성 회전이면 한 번 누를 때 일정 각도만 회전합니다.

***

### 6. 카메라가 지면과 가까워지면 수평각도 조절

“줌 인”으로 인해 카메라와 지면이 가까워졌다면, **Pitch**를 좀 더 낮춰(수평에 가깝게) 시네마틱 뷰를 만들고 싶을 수 있습니다.

* 구현 아이디어:
  1. **Tick** 이벤트에서 `SpringArm->TargetArmLength`를 확인
  2. 길이가 특정 임계값보다 작아지면, `SpringArm->SetRelativeRotation(FRotator(NewPitch, ..., ...))` 식으로 Pitch를 서서히 변화
  3. 다시 멀어지면 Pitch를 원상 복귀
* 부드럽게 만들려면:
  * **Interp(보간)** 을 사용: `CurrentPitch = FMath::FInterpTo(CurrentPitch, DesiredPitch, DeltaTime, InterpSpeed)`

#### 예시(블루프린트)

```plaintext
Event Tick (DeltaTime)
    CurrentLength = CameraSpringArm->TargetArmLength

    If (CurrentLength < 600) then
        // 지면과 근접: Pitch 낮춤
        DesiredPitch = -20 // (기본 -60에서 -20으로)
    Else
        DesiredPitch = -60 // 원래 탑다운 각도

    CurrentPitch = FMath::FInterpTo(CurrentPitch, DesiredPitch, DeltaTime, 3.0)
    CameraSpringArm->SetRelativeRotation(FRotator(CurrentPitch, CurrentYaw, 0))
```

***

### 7. 캐릭터 초상화 더블 클릭 시 초점 이동

1. \*\*UI(예: UMG)\*\*에서 캐릭터 초상화(또는 초상화 버튼)에 “Double Click” 이벤트를 처리합니다.
   * 그 이벤트가 발생하면, 카메라 Pawn에게 “해당 캐릭터 위치로 이동하라”는 명령을 보냅니다.
2. **카메라 Pawn**에서 “초점 이동” 함수를 만든다.
   * 예: `FocusOnActor(AActor* TargetActor)`
3. 내부 구현:
   1. 캐릭터의 현재 위치를 가져온다: `FVector TargetPos = TargetActor->GetActorLocation()`
   2. 카메라 Pawn 자신의 위치를 보간해서 이동(또는 순간이동).
      *   순간 이동:

          ```plaintext
          SetActorLocation(TargetPos + OffsetVector)
          ```
      * 보간 이동(부드럽게):
        * Tick에서 `CurrentLocation = FMath::VInterpTo(CurrentLocation, DesiredLocation, DeltaTime, InterpSpeed)` 같은 식
4. 만약 “회전도 캐릭터를 바라보게” 하고 싶다면, TargetActor의 위치를 기준으로 Yaw를 계산해서 회전도 보간해줄 수 있습니다.

***

### 8. 정리 & 팁

* **SpectatorPawn**을 쓰면 기본적으로 이동/회전 관련 Input 코드가 내장되어 있어서 편하지만, 직접 Pawn을 만들어도 무방합니다.
* **TopDownCharacter**는 이제 단순히 맵 상에서 움직이는 유닛(플레이어 아바타)으로 남아있고, 조작(입력)은 모두 카메라 Pawn이 담당합니다. 실제로 “발더스 게이트 3” 같은 파티 RPG에서는 보통 카메라가 유닛과 독립적으로 움직이고, 유닛을 선택/이동 명령만 주는 구조입니다.
* “카메라가 지면과 가까워지면 수평으로”는 꼭 필요한 것은 아니지만, 연출상 좋다면 위 예시처럼 `TargetArmLength`를 기준으로 Pitch를 보간해 주면 됩니다.
* Q/E 회전(또는 마우스 드래그 회전) 시, **카메라 Pawn**의 Yaw를 직접 변경하거나, SpringArm을 회전시켜도 됩니다. **단**, Roll은 0으로 고정해 흔들리지 않도록 관리하는 것이 보통입니다.
* 캐릭터 초상화 더블클릭 → 초점 이동은 **UI → 카메라 Pawn** 간 통신 로직이 핵심입니다. UMG에서 브로드캐스트 이벤트를 PlayerController나 Pawn에게 전달하고, Pawn이 이동하는 식입니다.

***

## 결론

1. **기존 TopDown 템플릿**의 “캐릭터 붙은 카메라” 구조를 버리고, **별도의 Camera Pawn**(또는 SpectatorPawn)을 만들어서 **독립적인 카메라 조작**을 구현합니다.
2. **카메라 이동/회전/줌** 및 **Pitch 보간** 등은 전부 이 Pawn 안에서 처리하며, “Q/E 회전”이나 “마우스 휠 줌” 등을 입력으로 받아 SpringArm 길이나 Pawn 회전을 갱신합니다.
3. **캐릭터 초상화 더블클릭 시** → “Pawn 위치를 캐릭터 위치로 부드럽게 보간 이동”하여 초점을 맞춥니다.

이렇게 구성하면, **발더스 게이트 3**와 비슷하게 **캐릭터와 분리된 자유로운 카메라**를 구현할 수 있습니다. 필요에 따라 **파티 중 다른 캐릭터를 선택해 초점 이동**하거나, **카메라 컷신** 등도 쉽게 처리 가능합니다.

</details>



결과물&#x20;

1. 발더스와 같은 방식
2. fps 와 무관하도록 구현함
3. 가려지는 오브젝트 처리하는 것은 아직 안 함
4. github 별도 브랜치로 관리
5. gpt 가 spectator pawn 쓰라고 한것때문에 삽질을 오래함. 그냥 pawn 으로 구현해야함. spectator pawn 에 있는 movement component 때문에 필요한 인풋이 작동안함. spectator 쓰려면cpp 로 상속받아 구현해야하는데 그정도까지 귀찮게 할일이 아니어서 pawn 으로 간단하게 끝 냄.
6. 여기까지 결과물 :&#x20;

{% embed url="https://github.com/SanggunLee/sg77s-game-dev/commit/4eeeb6af9d5d19ba62dbae64228935dd37cc71be" %}

TODO:

1. 카메라가 부드럽게 이동해야 한다
2. 케릭터를 가리는 오브젝트 부분을 투명화시키던지 해야 한다

{% embed url="https://www.youtube.com/live/Pl-0fdj4z3U" %}
