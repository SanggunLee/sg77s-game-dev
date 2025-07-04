# 세계 구현

<details>

<summary>게임 속 npc 들이 플레이어와 대화 할 때 llm 에 입력해야 하는 프롬프트용 텍스트 문서가 npc 마다 따로 관리되어야 하는 것이 있고, 모든 npc 가 공용으로 써야 하는 텍스트 데이터가 있어야 한다. 맞는가?</summary>

맞습니다!\
게임 속 NPC들이 LLM과 자연스럽고 세계관에 맞는 대화를 하기 위해서는 **프롬프트에 포함되는 텍스트 데이터**를 잘 분리하고 관리하는 것이 핵심입니다. 일반적으로 다음과 같이 나눌 수 있습니다:

***

#### ✅ 1. **NPC마다 따로 관리되어야 하는 프롬프트 텍스트**

각 NPC의 **개성**과 **역할**, **기억**, **지식 제한**, **세계관 내 위치** 등을 반영해야 합니다.

예:

* 이름, 나이, 직업, 말투, 성격
* 지금까지 플레이어와 나눈 대화 내용 (개별 메모리)
* 자신이 알고 있는 세계관 정보 (지형, 인물 관계 등)
* 자신이 모르는 정보 (예: 왕궁 밖 일은 모름)
* 진행 상황에 따른 상태 변화 (ex. 퀘스트 완료 여부)

```
NPC 이름: 엘라
직업: 숲의 약초상
성격: 조용하고 신중함
배경: 어릴 적 마녀에게 구출되어 약초학을 배움
현재 상황: 플레이어에게 치료약을 팔았음, 최근 숲에 이상한 기운을 느낌
```

***

#### ✅ 2. **모든 NPC가 공용으로 사용하는 프롬프트 텍스트**

모든 NPC가 동일하게 알아야 할 **세계관 정보**나 **기본 룰**, **언어 스타일**, **게임 내 시스템적인 규칙** 같은 것들을 포함합니다.

예:

* 세계관의 기본 설정 (왕국, 종족, 종교, 정치상황)
* 게임 내 시간 흐름, 계절
* 플레이어의 현재 상태 (레벨, 칭호 등)
* LLM이 따라야 할 말투 규칙 (ex. 고어체, 반말/존댓말, 감정 억제 등)

```
세계관 개요:
- 이 게임은 '크라운 월드'라는 대륙을 배경으로 한다.
- 주요 국가: 발데린 왕국, 리사노 공화국
- 현재는 리사노와 발데린의 국경 분쟁이 격화된 상태다.

대화 규칙:
- NPC는 플레이어에게 정중하게 말하되, 직업/성격에 따라 다르게 표현 가능
- 플레이어의 질문에 대해 알고 있는 범위 내에서만 답한다
```

***

#### 📁 폴더 구조 예시

```plaintext
LLM_Prompts/
├─ common/
│  ├─ world_lore.txt
│  ├─ dialogue_rules.txt
│  └─ player_status_template.txt
├─ npc_profiles/
│  ├─ ella_the_herbalist.txt
│  ├─ torak_the_guard.txt
│  └─ mira_the_scholar.txt
```

***

이런 구조로 분리하면 **대규모 게임의 확장성**도 확보되고, **LLM 프롬프트 용량 제한** 내에서 필요한 정보만 잘 선택해서 넣을 수 있어요.

원한다면 템플릿 문서도 같이 만들어 줄게요! &#x20;



좋습니다! 아래에 **NPC 프롬프트 구성용 템플릿 문서**와 **공용 프롬프트 템플릿 문서**를 나누어 만들어 드릴게요.

***

### 📄 1. `npc_profiles/ella_the_herbalist.txt` 예시 템플릿

```plaintext
[NPC 기본 정보]
이름: 엘라 (Ella)
나이: 32세
성별: 여성
종족: 인간
직업: 숲의 약초상

[성격]
조용하고 신중하며, 낯선 사람에게 경계심이 있음. 그러나 동물과 식물에 대해서 이야기할 때는 열정적임.

[말투/화법]
정중하고 차분한 말투. 단어 선택이 신중함. 자연, 약초와 관련된 은유를 자주 사용함.

[배경 이야기]
어릴 적 숲에서 길을 잃었을 때 마녀에게 구출되어 약초학을 배움. 이후 마을로 돌아와 약초 상점을 운영함. 마녀와의 관계는 비밀로 함.

[세계관 내 위치]
- 리버우드 마을 외곽의 작은 오두막에 거주.
- 왕국 전쟁에는 관심이 없지만, 숲에 일어나는 이상현상에 민감함.

[NPC 기억 (대화 히스토리 요약)]
- 플레이어에게 "상처 치료약"을 2회 판매함.
- 플레이어가 "숲의 동굴에 이상한 기운이 난다"고 말했음.
- 플레이어에게 조심하라고 경고함.

[알고 있는 정보]
- 숲의 생물과 식물 종류, 독초 구분법
- 근처 지역의 날씨 변화 및 숲의 이상 징후
- 마을 사람들 중 일부의 개인적 성향

[알 수 없는 정보]
- 왕국의 정치 상황
- 도시 내 사건들
- 플레이어의 과거 행적
```

***

### 📄 2. `common/world_lore.txt` 예시 템플릿

```plaintext
[세계관 기본 설정]
- 대륙 이름: 크라운 월드 (Crown World)
- 주요 세력: 발데린 왕국, 리사노 공화국, 중립지대 '회색 초원'
- 현재 상황: 발데린과 리사노의 국경 분쟁이 격화되고 있음
- 고대 유적과 마법은 일부 존재하나 대부분 잊혀짐
- 마법은 위험한 기술로 인식됨

[세계의 규칙]
- 죽음은 되돌릴 수 없는 것으로 여겨짐 (부활 불가)
- 계절과 시간 흐름은 게임 내 현실과 비슷함
- 말과 행동은 NPC의 기억에 영향을 줄 수 있음

[기본 대화 규칙]
- NPC는 자신이 직접 겪은 일이나 확실히 들은 일만 말할 수 있음
- 모든 NPC는 세계관에서 허용된 말투와 정보만 사용할 수 있음
- 플레이어의 지위, 행동에 따라 말투나 태도가 달라질 수 있음

[중요 키워드 목록]
- "에테르석" = 고대 마법의 원천
- "성검 아르타이르" = 전설의 무기
- "붉은 달의 밤" = 재앙이 일어났던 사건
```

***

### 📄 3. `common/player_status_template.txt` 예시

```plaintext
[플레이어 상태 정보]
이름: ${player_name}
종족: ${player_race}
직업: ${player_class}
현재 레벨: ${player_level}
명성: ${player_reputation}
현재 위치: ${player_location}

[진행 중 퀘스트]
- ${quest_1_title}: 상태 = ${quest_1_status}
- ${quest_2_title}: 상태 = ${quest_2_status}

[플레이어와의 관계]
- 이 NPC는 플레이어에 대해 ${npc_opinion}를 가지고 있음
- 최근 플레이어의 행동으로 인해 감정 변화가 ${emotion_change}
```

***

필요하다면 이 템플릿을 기반으로 자동으로 `.txt` 생성해주는 툴이나 언리얼 엔진에서 불러올 수 있는 구조도 같이 도와줄 수 있어요!\
지금 어느 방향으로 더 이어서 도와드릴까요?

* 🔧 템플릿 자동 생성 스크립트
* 🎮 언리얼 엔진 연동 방법
* 🧠 대화 로그 저장/로드 시스템 설계
* 🗃️ LLM 프롬프트 구성 자동화

원하는 방향 골라줘도 되고, 다 도와줄 수 있어요!

</details>

<details>

<summary>언리얼5에서 C++ 로 구현하기. llm 프롬프트 생성 후 통신</summary>



좋습니다!\
CRPG 스타일 게임에서 \*\*NPC 대사를 LLM API(OpenAI 등)\*\*로 생성하려면 언리얼 엔진에서 다음과 같은 시스템을 구성해야 합니다.

***

### 🎯 목표 요약

* 언리얼 엔진 5에서 NPC 클릭 → 대화 창 열림
* 대화 요청 → LLM API 호출 → 응답 수신
* NPC 개별 프롬프트 + 공용 프롬프트 기반 생성
* 대화 결과 저장 (NPC 별 기억 유지)
* 대화 UI에 결과 출력

***

### 🧩 시스템 구성도 (요약)

```plaintext
[Player] 클릭
   ↓
[NPC Blueprint] → "TalkToNPC"
   ↓
[Prompt Generator (C++)]
   ↓
[HTTP 요청 → LLM API (OpenAI etc)]
   ↓
[LLM 응답 수신 → JSON 파싱]
   ↓
[대화 UI 업데이트]
   ↓
[대화 로그 저장 (JSON or SaveGame)]
```

***

### 🔧 언리얼 엔진 연동 순서

#### ① HTTP 요청 설정

LLM API는 일반적인 **REST API (POST)** 형식이므로 언리얼의 `HTTP Module`로 접근 가능.

**📌 `Build.cs`에 추가:**

```csharp
PublicDependencyModuleNames.AddRange(new string[] { "HTTP", "Json", "JsonUtilities" });
```

**📌 헤더 포함:**

```cpp
#include "HttpModule.h"
#include "Http.h"
#include "Interfaces/IHttpRequest.h"
#include "Interfaces/IHttpResponse.h"
```

***

#### ② LLM 요청 생성 함수

```cpp
void UNPCDialogueComponent::SendPromptToLLM(FString Prompt)
{
    TSharedRef<IHttpRequest, ESPMode::ThreadSafe> Request = FHttpModule::Get().CreateRequest();
    Request->SetURL("https://api.openai.com/v1/chat/completions");
    Request->SetVerb("POST");
    Request->SetHeader("Content-Type", "application/json");
    Request->SetHeader("Authorization", "Bearer YOUR_API_KEY");

    FString Body = R"(
    {
        "model": "gpt-4",
        "messages": [
            {"role": "system", "content": "세계관 설정"},
            {"role": "user", "content": ")" + Prompt + R"("}
        ],
        "temperature": 0.7
    })";

    Request->SetContentAsString(Body);
    Request->OnProcessRequestComplete().BindUObject(this, &UNPCDialogueComponent::OnResponseReceived);
    Request->ProcessRequest();
}
```

***

#### ③ 응답 처리 함수

```cpp
void UNPCDialogueComponent::OnResponseReceived(FHttpRequestPtr Request, FHttpResponsePtr Response, bool bWasSuccessful)
{
    if (bWasSuccessful && Response->GetResponseCode() == 200)
    {
        FString ResponseString = Response->GetContentAsString();
        TSharedPtr<FJsonObject> JsonResponse;
        TSharedRef<TJsonReader<>> Reader = TJsonReaderFactory<>::Create(ResponseString);

        if (FJsonSerializer::Deserialize(Reader, JsonResponse))
        {
            FString Message = JsonResponse->GetArrayField("choices")[0]->AsObject()->GetObjectField("message")->GetStringField("content");
            UE_LOG(LogTemp, Warning, TEXT("LLM 응답: %s"), *Message);

            // 대화 UI에 출력
            OnLLMResponse.Broadcast(Message);
        }
    }
    else
    {
        UE_LOG(LogTemp, Error, TEXT("LLM API 호출 실패"));
    }
}
```

***

#### ④ NPC마다 다른 프롬프트 입력

NPC마다 저장된 `.txt` 또는 `DataAsset`, 혹은 `USTRUCT` 구조체로 프롬프트를 저장해두고 LLM 호출 전 병합합니다.

```cpp
FString FinalPrompt = CommonPrompt + "\n" + NPCProfile + "\n" + PlayerContext;
SendPromptToLLM(FinalPrompt);
```

***

#### ⑤ 대화 내용 저장 및 기억 시스템

* `SaveGame` 객체에 저장
* `TMap<FName, TArray<FDialogueLog>>` 형태로 NPC 별 대화 저장
* 다음 대화 시 이전 로그를 프롬프트에 포함

***

### 🎮 언리얼 블루프린트 연동 예시

* NPC Actor에 `NPCDialogueComponent` 추가
* 블루프린트에서 `SendPromptToLLM` 노드 호출
* `OnLLMResponse` 델리게이트 바인딩 → UI 업데이트

***

### ✨ 확장 팁

| 기능    | 구현 아이디어                                |
| ----- | -------------------------------------- |
| 감정 변화 | 대화 결과에서 감정 키워드 추출 후 `NPCEmotion` 상태 갱신 |
| 지식 제한 | 프롬프트에 `"이 NPC는 왕국 바깥일은 전혀 모른다."` 포함    |
| 성격 반영 | `"이 NPC는 말이 짧고 까칠하다."` 포함              |
| 감정 분석 | LLM에게 `"이 문장에서 플레이어에게 느낀 감정을 추출해줘"` 요청 |

***

필요하다면:

* 📁 `Prompt 구성 자동화 시스템`
* 🧠 `대화 로그를 파일로 저장/불러오는 SaveGame 클래스`
* 📊 `대화 UI 템플릿`

도 바로 만들어 드릴 수 있어요. 어떤 부분부터 코드나 예제 추가해드릴까요?

</details>

<details>

<summary>npc 가 LLM 으로 대화 가능하게 하는 플러그인<br>LLM 자동 코드 생성을 위한 문서</summary>

## LLM Dialogue Plugin 설계 및 명세 문서

이 문서는 Unreal Engine 5 기반 게임에서 NPC 대사 생성을 위해 LLM API(OpenAI GPT 등)를 연동하는 **LLM Dialogue Plugin**의 아키텍처 및 함수 설계 명세입니다. 이 문서를 기반으로 LLM이 자동으로 C++ 코드 및 관련 블루프린트 연동을 생성할 수 있도록 구성되어 있습니다.

***

### 개요

* **플러그인 이름:** `LLMDialoguePlugin`
* **목적:** 게임 내 NPC가 플레이어와 상호작용할 때, LLM API를 통해 대사를 실시간 생성하도록 지원
* **기반 엔진:** Unreal Engine 5
* **주요 언어:** C++

***

### 아키텍처 구성도

```plaintext
[Player] 클릭
   ↓
[NPC Actor] + [LLMDialogueComponent]
   ↓
[Prompt 구성 함수] + [HTTP 요청 처리 함수]
   ↓
[LLM API (ex: GPT-4)]
   ↓
[응답 수신] → [Blueprint Event로 전달]
   ↓
[UI 출력 및 대화 로그 저장]
```

***

### 주요 컴포넌트

#### 1. `ULLMDialogueComponent`

> NPC Actor에 부착 가능한 ActorComponent. LLM 요청과 응답 처리를 담당.

**주요 함수 및 변수**

**`void SendPromptToLLM(const FString& Prompt)`**

* **역할:** 전달된 프롬프트를 LLM API에 POST 방식으로 전송
* **입력:** `Prompt` (대사 생성을 위한 전체 텍스트)
* **출력:** 없음 (결과는 콜백을 통해 전달)

**`void OnResponseReceived(...)`**

* **역할:** HTTP 응답이 완료되었을 때 호출되며, JSON에서 메시지를 추출하여 이벤트로 전달
* **출력:** `FString Message` (LLM 응답 텍스트)

**`FOnLLMResponse OnLLMResponse`**

* **역할:** LLM 응답을 블루프린트에 전달하는 delegate
* **BlueprintAssignable:** UI 연동에 사용

***

### API 요청 사양 (OpenAI GPT 예시)

#### URL

```
https://api.openai.com/v1/chat/completions
```

#### 요청 Body 예시

```json
{
  "model": "gpt-4",
  "messages": [
    {"role": "system", "content": "세계관 설명 및 말투 규칙 등"},
    {"role": "user", "content": "NPC 프롬프트 텍스트"}
  ],
  "temperature": 0.7
}
```

#### 요청 헤더

* `Content-Type: application/json`
* `Authorization: Bearer <YOUR_API_KEY>`

***

### 사용 예 (블루프린트)

1. NPC에 `LLMDialogueComponent` 부착
2. `SendPromptToLLM(PromptString)` 호출
3. `OnLLMResponse` 바인딩 → 텍스트 출력 위젯에 연결

***

### 확장 가능 항목

| 기능                  | 설명                         |
| ------------------- | -------------------------- |
| SaveGame 연동         | NPC별 대화 로그를 저장하여 기억 유지     |
| 다양한 LLM 연동          | Claude, Gemini 등으로 확장 가능   |
| Prompt Template 시스템 | JSON 기반 템플릿 구성 자동화         |
| 프롬프트 큐              | 다수의 NPC 요청을 순차적으로 처리하는 대기열 |
| 감정 분석               | 대사에 포함된 감정 태그 추출 가능        |

***

### 파일 구조 예시

```plaintext
LLMDialoguePlugin/
├─ Source/LLMDialoguePlugin/
│  ├─ Public/
│  │  └─ LLMDialogueComponent.h
│  └─ Private/
│     └─ LLMDialogueComponent.cpp
├─ Resources/
│  └─ Icon128.png
├─ LLMDialoguePlugin.uplugin
```

***

### 향후 발전 방향

* DialogueStateMachine과의 통합
* NPC별 프로필 파일 자동 로딩 (DataTable 또는 JSON)
* 멀티 캐릭터 동시 대화 지원
* 프롬프트 내 세계관 정보 병합 최적화

***

이 문서를 기반으로 LLM은 C++ 코드 생성, 블루프린트 노드 생성, HTTP API 자동화 등을 수행할 수 있습니다.

</details>
