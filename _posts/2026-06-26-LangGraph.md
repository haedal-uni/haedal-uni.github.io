---
categories: RAG
tags: [RAG, LangChain, LangGraph]
---

# LangGraph로 RAG 챗봇 고도화하기

이전 글에서 기본적인 RAG 챗봇을 만들었다.  

사용자가 질문하면 문서를 검색하고, LLM이 답변을 생성하는 단순한 흐름이었다.

그런데 실제로 사용해보니 한계가 있었다.

<br>

- "겐지 카운터 알려줘" → "모이라 어때?" 같은 후속 질문을 이해하지 못함 (대화 흐름 기억 없음)

- "솔저: 76으로 어떻게 해야 해?" → 힐러를 추천하는 오류 (역할 고정 모드 미반영)

- 스탯(킬, 딜량 등)을 입력해도 단순히 텍스트로만 처리

<br>

이 문제들을 해결하기 위해 선택한 방법이 **LangGraph**다.

<br><br><br>

---

# LangGraph란?

**LangGraph**는 LLM 애플리케이션의 처리 흐름을 **노드(Node)와 엣지(Edge)로 구성된 그래프**로 표현하는 프레임워크다.

기존 LCEL 체인이 `A → B → C`처럼 선형으로 흐른다면, LangGraph는 조건에 따라 분기하거나 특정 단계를 건너뛸 수 있다.

```
LCEL:   입력 → 프롬프트 → LLM → 파서 → 출력  (항상 동일한 경로)

LangGraph:
        입력 → 검증 → 스탯파싱 → 문맥분석 → 검색 → 전략판단 → 답변생성 → 출력
                         ↓ (오류)               ↓ (역할 질문 필요)
                       출력                   역할선택 UI 표시
```

<br>

핵심 개념은 세 가지다.

| 개념 | 설명 |
|------|------|
| **State** | 노드 간에 공유되는 데이터 묶음. 각 노드는 State를 읽고 수정한 결과를 반환한다 |
| **Node** | 실제 작업을 수행하는 함수. State를 받아 State의 일부를 반환한다 |
| **Edge** | 노드 간의 연결. 조건에 따라 다음 노드를 다르게 선택하는 **conditional edge**를 지원한다 |

<br><br><br>

---

# 전체 구조

기존 단일 파일 구조에서 역할별로 파일을 분리했다.

```
chat/
├── rag_class.py          # 문서 로딩, 임베딩, VectorDB, LLM 초기화
├── chatbot_service.py    # 싱글톤 컴포넌트 관리 (초기화 비용 절감)
├── chatbot_graph.py      # LangGraph 그래프 정의 (핵심 로직)
└── views.py              # Django API — 세션 관리 + 그래프 실행
```

<br>

요청 하나가 처리되는 전체 흐름은 다음과 같다.

```
사용자 요청
    ↓
views.py — 세션에서 대화 컨텍스트를 꺼내 그래프에 전달
    ↓
chatbot_graph.py — 11개 노드를 순서대로 실행
    ↓
chatbot_service.py — 각 노드가 필요할 때 LLM/Retriever를 꺼내 씀
    ↓
views.py — 결과를 세션에 저장 후 JSON 응답
```

<br><br><br>

---

# rag_class.py — 문서 로딩 & 컴포넌트 초기화

기존 `ChatBot` 클래스에서 문서 분할 방식 하나가 바뀌었다.

<br>

## 2단계 분할 전략

이전에는 `RecursiveCharacterTextSplitter`만 사용했는데, 이번에는 **두 단계로 나눠서** 처리한다.

```py
# 1단계: 마크다운 헤더 기준으로 먼저 나눈다
header_splitter = MarkdownHeaderTextSplitter(
    headers_to_split_on=[
        ("#", "H1"),
        ("##", "H2"),
        ("###", "H3"),
    ],
    strip_headers=False,
)
header_docs = header_splitter.split_text(md_text)

# 2단계: 헤더로 나눈 덩어리를 글자 수 기준으로 다시 자른다
char_splitter = RecursiveCharacterTextSplitter(
    chunk_size=self.chunk_size,
    chunk_overlap=self.chunk_overlap,
    separators=["\n\n", "\n", " ", ""],
)
docs = char_splitter.split_documents(header_docs)
```

<br>

이렇게 하면 각 chunk의 `metadata`에 헤더 정보(`H1`, `H2`, `H3`)가 자동으로 붙는다.

예를 들어 "라인하르트 운영법" 섹션에서 자른 chunk라면 메타데이터가 이렇게 된다.

```json
{
  "H1": "탱커",
  "H2": "라인하르트",
  "H3": "운영법"
}
```

덕분에 Retriever가 어떤 chunk를 참고했는지 계층적으로 추적할 수 있다.

<br><br><br>

---

# chatbot_service.py — 싱글톤 컴포넌트 관리

LangGraph의 노드들은 각각 독립적인 함수다. 

매번 호출될 때마다 LLM이나 Retriever를 새로 만들면 초기화 비용이 크다.

이 문제를 해결하기 위해 별도 파일로 싱글톤 관리 모듈을 분리했다.

```py
_chatbot: Optional[ChatBot] = None
_retriever: Optional[Any] = None
_llm: Optional[Any] = None

def initialize_chatbot() -> Tuple[ChatBot, Any, Any]:
    global _chatbot, _retriever, _llm

    if _chatbot is not None and _retriever is not None and _llm is not None:
        return _chatbot, _retriever, _llm  # 이미 초기화됐으면 재사용

    bot = ChatBot()
    retriever = bot.build_rag_components()
    llm = bot.get_llm()

    _chatbot = bot
    _retriever = retriever
    _llm = llm

    return _chatbot, _retriever, _llm

def get_chatbot_components() -> Tuple[ChatBot, Any, Any]:
    """LangGraph 노드에서 사용하는 공용 getter."""
    return initialize_chatbot()
```

<br>

모든 노드는 `get_chatbot_components()`를 호출해서 이미 만들어진 컴포넌트를 가져다 쓴다. 

무거운 초기화 작업은 서버 시작 후 단 한 번만 일어난다.

<br><br><br>

---

# chatbot_graph.py — LangGraph 핵심 로직

가장 핵심이 되는 파일로 11개의 노드와 그 연결 관계를 정의한다.

<br>

## State 정의

노드 간에 공유되는 데이터 구조를 먼저 정의한다. TypedDict를 사용해 타입을 명시했다.

```py
class ChatbotGraphState(TypedDict, total=False):
    message: str                          # 사용자 메시지
    conversation_context: Dict[str, Any]  # 이전 대화 컨텍스트 (세션에서 전달)
    intent: Optional[str]                 # 질문 의도 (counter/swap/stay 등)
    current_hero: Optional[str]           # 사용자가 플레이 중인 영웅
    target_enemy: Optional[str]           # 카운터 대상 적 영웅
    role_filter: Optional[str]            # 역할 필터 (tank/damage/support)
    retrieved_docs: List[Dict]            # 검색된 문서 chunk 목록
    answer: str                           # 최종 답변
    context_patch: Dict[str, Any]         # 세션에 저장할 컨텍스트 변경분
    result: Dict[str, Any]                # 최종 응답 딕셔너리
    # ... 그 외 다수
```

`total=False`로 선언했기 때문에 모든 키는 선택적이다. 

각 노드는 자신이 처리한 키만 반환하면 된다. LangGraph가 기존 State에 병합해준다.

<br><br>

## 그래프 흐름

```
START
  ↓
validate_input          — 입력값 유효성 검사
  ↓
parse_stats             — 스탯 텍스트 파싱 (킬, 딜량 등)
  ↓
llm_parse_context       — LLM으로 의도·영웅·적 추출
  ↓
merge_context           — 규칙 기반 + LLM 결과 통합, 세션 컨텍스트 병합
  ↓ (역할 질문 필요?)
clarify_role_filter  ←→  build_retrieval_queries
  ↓                              ↓
format_response          retrieve_docs — 검색 쿼리 생성 후 문서 검색
                                 ↓
                          judge_strategy — 전략 판단 (추천 영웅, 추천 유형)
                                 ↓
                          generate_answer — 최종 답변 생성
                                 ↓
                    generate_suggested_questions — 추천 후속 질문 생성
                                 ↓
                          format_response — 응답 포맷팅
                                 ↓
                               END
```

<br><br>

## 주요 노드 설명

### 1. validate_input_node — 입력값 검사

가장 먼저 실행된다. 메시지가 비어있으면 즉시 에러를 State에 기록하고 이후 노드들은 에러 여부를 확인해 스킵한다.

```py
def validate_input_node(state: ChatbotGraphState) -> ChatbotGraphState:
    message = state.get("message", "").strip()
    role_filter = state.get("role_filter")

    if not message and not role_filter:
        return {"error": "질문을 입력해주세요."}

    return {"message": message, "need_clarification": False}
```

<br><br>

### 2. parse_stats_from_text_node — 스탯 파싱

사용자가 "리퍼 킬 5 딜 8000 데스 3" 같은 자유 형식으로 스탯을 입력하면, LLM을 활용해 구조화된 데이터로 변환한다.

```py
def detect_stat_input(message: str) -> bool:
    stat_keywords = ["킬", "데스", "딜", "힐", "도움", "어시", "사망", "피해", "치유"]
    has_keyword = any(kw in message for kw in stat_keywords)
    has_number = bool(re.search(r"\d{2,}", message))
    return has_keyword and has_number
```

스탯 키워드와 숫자가 함께 등장할 때만 파싱을 시도한다. 

LLM에게 JSON 형식으로만 답하도록 프롬프트를 설계해 구조화된 데이터를 얻는다.

```json
{
  "enemy_stats": {"라마트라": {"kills": 8, "damage": 12000}},
  "my_stats":    {"에코": {"kills": 5, "deaths": 3, "damage": 9500}},
  "my_team_stats": {}
}
```

파싱된 데이터는 이후 `judge_strategy_node`와 `generate_answer_node`에서 

"어떤 적이 가장 위협적인지", "내 딜량이 낮은 이유가 무엇인지" 분석하는 데 쓰인다.

<br><br>

### 3. llm_parse_context_node — LLM 기반 문맥 분석

사용자의 메시지에서 의도(intent), 현재 영웅, 적 영웅 등을 LLM이 추출한다.

```py
# LLM에게 아래 형식으로만 답하도록 요청
{
  "intent": "counter | swap | stay | performance_improve | map_strategy | general",
  "current_hero": "현재 플레이 중인 영웅 또는 null",
  "current_hero_role": "tank | damage | support 또는 null",
  "target_enemy": "카운터 대상 적 영웅 또는 null",
  "enemy_team": ["적팀 영웅1", "적팀 영웅2"]
}
```

<br>

의도는 6가지로 분류된다.

| intent | 설명 |
|--------|------|
| `counter` | "겐지 카운터 알려줘" 같은 상대 영웅 대응법 질문 |
| `swap` | "다른 영웅으로 바꿔야 할까?" 같은 교체 고민 |
| `stay` | "이 영웅 계속 써도 돼?" 같은 유지 여부 질문 |
| `performance_improve` | "딜량 어떻게 올려?" 같은 성능 개선 질문 |
| `map_strategy` | 맵·공수 관련 전략 질문 |
| `general` | 위에 해당하지 않는 일반 질문 |

<br>

**원문 검증 단계**가 중요하다. 

LLM이 추출한 영웅 이름이 실제 메시지 원문에 등장하는지 반드시 확인한다. 

LLM은 "짐작"으로 채울 수 있기 때문이다.   

```py
# target_enemy가 이번 메시지 원문에 실제로 존재할 때만 신뢰
if (normalized_enemy in [normalize_hero_name(h) for h in HEROES]
        and normalized_enemy in message):
    result["llm_target_enemy"] = normalized_enemy
```

<br><br>

### 4. merge_context_node — 컨텍스트 통합

LangGraph에서 가장 복잡한 노드다. 세 가지 정보를 통합해 최종 컨텍스트를 결정한다.

```
LLM 파싱 결과 + 규칙 기반 추출 결과 + 이전 세션 컨텍스트 → 최종 State
```

핵심 로직들을 하나씩 살펴보면 다음과 같다.

<br>

**세션 타임아웃**

오버워치 한 판은 보통 10~20분이다. 

마지막 메시지로부터 10분이 지나면 새 게임으로 간주하고 컨텍스트를 초기화한다.

```py
SESSION_TIMEOUT_SECONDS = 10 * 60
if last_message_ts and (now_ts - last_message_ts) > SESSION_TIMEOUT_SECONDS:
    context = {}  # 컨텍스트 초기화
```

<br>

**적 컨텍스트 자동 소멸**

이전 대화에서 언급된 적 영웅이 세션에 눌어붙는 문제가 있었다.

2턴 연속으로 적 영웅이 언급되지 않으면 자동으로 비운다.

```py
STALE_ENEMY_TURN_LIMIT = 2
if no_enemy_turn_count >= STALE_ENEMY_TURN_LIMIT:
    # target_enemy, enemy_team 세션에서 제거
    context = {k: v for k, v in context.items()
               if k not in ("target_enemy", "enemy_team", "high_threat_enemy")}
```

<br>

**역할 필터 우선순위**

```
1순위: 이번 메시지에서 명시적으로 요청한 필터 ("탱커로 추천해줘")
2순위: 현재 플레이 중인 영웅의 실제 역할
3순위: 세션에 남아있는 이전 role_filter (최후 수단)
```

2순위가 3순위보다 높아야 한다. 

그렇지 않으면 예전에 "탱커 추천해줘"라고 물었던 필터가 세션에 박혀 

딜러로 영웅을 바꿨는데도 탱커만 추천하는 문제가 발생한다. 

<br><br>

### 5. clarify_role_filter_node — 역할 선택 UI

카운터 영웅을 물어봤는데 역할 필터가 없을 때 발동한다. 답변 대신 버튼 UI를 반환해 사용자가 역할을 먼저 선택하게 한다.

```py
def clarify_role_filter_node(state: ChatbotGraphState) -> ChatbotGraphState:
    answer = f"{target_enemy}를 카운터하는 영웅을 어떤 역할 기준으로 볼까요?"
    choice_buttons = [
        {"label": "전체", "value": "all", "type": "role_filter"},
        {"label": "탱커", "value": "tank", "type": "role_filter"},
        {"label": "딜러", "value": "damage", "type": "role_filter"},
        {"label": "힐러", "value": "support", "type": "role_filter"},
    ]
    return {"answer": answer, "choice_buttons": choice_buttons, ...}
```

사용자가 버튼을 선택하면 `role_filter` 값이 담긴 요청이 다시 들어오고, 그때 `build_retrieval_queries_node`로 분기된다.

<br><br>

### 6. build_retrieval_queries_node — 검색 쿼리 생성

단일 검색 대신 **의도에 맞는 여러 쿼리**를 생성해 검색 품질을 높인다.

```py
queries = [message]  # 기본: 원문 그대로

if target_enemy and enemy_named_this_turn:
    queries.append(f"{target_enemy} 카운터 상대법 견제 방법")
    queries.append(f"{target_enemy} 약점 스킬 무력화")

if current_hero:
    queries.append(f"{current_hero} 운영법 스킬 사용법")

if map_name:
    queries.append(f"{map_name} {side_text} 맵 운영법 포지션 영웅")
    if target_enemy and enemy_named_this_turn:
        queries.append(f"{map_name} {side_text} {target_enemy} 대응법")
```

`enemy_named_this_turn` 플래그가 중요하다. 

이번 메시지에서 실제로 언급된 적만 쿼리에 포함한다. 

그렇지 않으면 이전 대화의 적이 이번 검색 결과를 오염시킨다.

<br><br>

### 7. retrieve_docs_node — 문서 검색

여러 쿼리로 각각 검색한 결과를 중복 제거 후 합친다.

```py
seen_contents: set = set()
for query in state.get("retrieval_queries", []):
    docs = retrieve_documents(retriever, query)
    for doc in docs:
        content_key = content[:500]
        if content_key in seen_contents:
            continue  # 중복 chunk 제거
        seen_contents.add(content_key)
        all_docs.append(doc_dict)

all_docs = all_docs[:12]  # 최대 12개 chunk
```

각 chunk에는 `doc_id`가 붙어, 이후 `generate_answer_node`에서 LLM이 실제로 참고한 문서 ID를 추적할 수 있다.

<br><br>

### 8. judge_strategy_node — 전략 판단

LLM에게 "어떤 영웅을 추천할지"를 별도로 먼저 판단시킨다. 최종 답변 생성과 역할을 분리한 것이다.

```py
# LLM이 반환하는 형식
{
  "recommendation_type": "counter_pick | hero_swap | stay_and_adapt | ...",
  "recommended_heroes": ["모이라", "키리코"],
  "strategy_reason": "라마트라의 공허 방벽을 관통하는 영웅이 유리함"
}
```

판단 후에는 역할 필터 검증을 한 번 더 거친다. LLM이 허용 범위 밖의 영웅을 추천했다면 걸러낸다.

```py
if allowed_hero_set is not None:
    filtered = [h for h in recommended_heroes if h in allowed_hero_set]
    recommended_heroes = filtered
```

<br><br>

### 9. generate_answer_node — 최종 답변 생성

전략 판단 결과를 바탕으로 실제 사용자에게 보여줄 답변을 생성한다. 

프롬프트에는 역할 고정 규칙이 명시적으로 포함된다.

```
=== 최우선 규칙 (절대 위반 금지) ===
오버워치는 역할 고정(Role Lock) 모드를 운영한다.
딜러로 입장하면 그 판에서는 딜러 영웅만 선택 가능하다.

현재 사용자 영웅: 에코 (역할: 딜러)
영웅 교체 추천은 반드시 딜러 역할만: 겐지, 트레이서, 솜브라 ...
```

답변은 JSON 형식으로 반환받아 파싱한다. 

LLM이 마크다운 기호를 섞어 쓰는 경우를 대비해 `sanitize_answer_for_user()`로 후처리한다.

```py
def sanitize_answer_for_user(answer: str) -> str:
    # "[문서 1]에서 언급했듯이" 같은 출처 표시 제거
    sanitized = re.sub(r"\s*\[문서\s*\d+\][^,.\n]{0,12}(?:듯이|면서)?,?", "", sanitized)
    # **볼드** → 볼드 (마크다운 제거)
    sanitized = re.sub(r"\*\*(.+?)\*\*", r"\1", sanitized)
    # 글머리 기호 제거
    sanitized = re.sub(r"^\s*[\*\-]\s+", "", sanitized, flags=re.MULTILINE)
    return sanitized.strip()
```

<br><br>

### 10. generate_suggested_questions_node — 후속 질문 추천

답변 아래에 표시할 "빠른 질문 버튼" 3개를 생성한다. 

LLM이 실패하면 인텐트 기반 규칙으로 폴백한다.

```py
def build_fallback_suggested_questions(state: ChatbotGraphState) -> List[str]:
    if state.get("has_stats"):
        return ["데스 줄이는 법 알려줘", "딜량 더 올리는 방법은?", "영웅 바꿔야 해?"]
    if target_enemy:
        return [f"{target_enemy} 피해야 할 행동은?", f"{target_enemy} 카운터 영웅", ...]
    ...
```

<br><br>

### 11. format_response_node — 응답 포맷팅

모든 노드의 결과를 하나의 JSON 응답으로 정리한다.

```py
return {
    "result": {
        "answer": answer,
        "intent": state.get("intent"),
        "recommendation_type": state.get("recommendation_type"),
        "recommended_heroes": state.get("recommended_heroes", []),
        "suggested_questions": state.get("suggested_questions", []),
        "choice_buttons": state.get("choice_buttons", []),
        "context_patch": state.get("context_patch", {}),
        "has_stats": state.get("has_stats", False),
    }
}
```

<br><br>

## 그래프 조립

```py
def build_chatbot_graph():
    graph = StateGraph(ChatbotGraphState)

    # 노드 등록
    graph.add_node("validate_input", validate_input_node)
    graph.add_node("parse_stats", parse_stats_from_text_node)
    # ... 이하 생략

    # 엣지 연결
    graph.add_edge(START, "validate_input")
    graph.add_conditional_edges(
        "validate_input",
        route_after_validation,
        {"parse_stats": "parse_stats", "format_response": "format_response"}
    )
    # ... 이하 생략

    return graph.compile()
```

`add_conditional_edges`는 라우팅 함수의 반환값을 보고 다음 노드를 결정한다. 

에러가 있으면 `format_response`로 바로 이동하고, 정상이면 다음 단계로 진행하는 패턴이 반복된다.

<br><br>

---

# views.py — 세션 관리

기존 뷰에서 달라진 핵심은 **Django 세션으로 대화 컨텍스트를 관리**한다는 점이다.

```py
@csrf_exempt
@require_http_methods(["POST"])
def chat_api(request):
    data = json.loads(request.body or b"{}")
    message = str(data.get("message", "")).strip()
    role_filter = data.get("role_filter") or None

    # 대화 초기화 요청
    if data.get("reset"):
        request.session.pop("coach_context", None)
        return JsonResponse({"ok": True})

    # 세션에서 이전 컨텍스트를 꺼내 그래프에 전달
    conversation_context = {
        **(request.session.get("coach_context", {}) or {}),
        **extra_context,  # 프론트엔드에서 추가로 보낸 컨텍스트
    }

    # LangGraph 실행
    graph_result = run_chatbot_graph(
        message=message,
        conversation_context=conversation_context,
        role_filter=role_filter,
    )

    # 그래프 결과의 context_patch를 세션에 반영
    context_patch = result.pop("context_patch", {}) or {}
    updated_context = {**base_context, **context_patch}
    request.session["coach_context"] = updated_context

    return JsonResponse(result)
```

**context_patch** 패턴이 핵심이다. 

그래프가 "변경된 것만" `context_patch`로 반환하면, 뷰에서 기존 세션에 덮어씌운다. 

덕분에 변경되지 않은 값들은 세션에 그대로 남는다.

예를 들어 "겐지 카운터 알려줘"라고 물으면 `target_enemy: "겐지"`가 세션에 저장되고 

바로 다음에 "운영법도 알려줘"라고 물어도 겐지 컨텍스트가 이어진다.

<br><br>

---

# 마무리

전체 흐름을 다시 정리하면 다음과 같다.

```
① Django 세션에서 이전 대화 컨텍스트를 꺼냄
② validate → 스탯 파싱 → LLM 문맥 분석 → 컨텍스트 통합
③ 역할 질문이 필요하면 UI 버튼 반환, 아니면 다음 단계로
④ 의도별 검색 쿼리 생성 → 문서 검색 → 전략 판단
⑤ 최종 답변 생성 → 후속 질문 버튼 생성 → 응답 포맷팅
⑥ context_patch를 Django 세션에 반영
```

LangGraph를 적용하면서 얻은 가장 큰 이점은 **각 단계를 독립적으로 수정할 수 있다**는 점이다. 

스탯 파싱 로직을 바꾸고 싶으면 `parse_stats_from_text_node`만 수정하면 되고 

전략 판단 방식을 바꾸고 싶으면 `judge_strategy_node`만 건드리면 된다.

다음에는 이 구조를 기반으로 대화 기록을 더 정교하게 관리하거나, 멀티 에이전트 구조로 확장하는 작업을 해볼 예정이다.

<br><br><br>

---

# 부록: 트러블슈팅

개발 과정에서 반복적으로 마주친 문제들과 해결 방법을 정리했다.

<br>

## 1. LLM이 이전 대화의 적 영웅을 다음 질문에 그대로 가져오는 문제

**증상**: "겐지 카운터 알려줘" 후 "운영법 알려줘"라고 하면, 두 번째 질문에서도 겐지가 target_enemy로 박혀있어 엉뚱한 답변이 나왔다.

**원인**: LLM이 컨텍스트를 보고 이전 대화의 `target_enemy`를 그대로 이어받는 경향이 있었다.

**해결**: `enemy_named_this_turn` 플래그를 도입했다. 

이번 메시지 원문에 적 영웅 이름이 실제로 등장했을 때만 `True`로 설정되며 

검색 쿼리 생성과 답변 프롬프트 모두 이 플래그를 기준으로 적 정보를 포함할지 결정한다. 

추가로 2턴 연속으로 적이 언급되지 않으면 세션에서 자동으로 비우는 소멸 로직도 함께 적용했다.

<br><br>

## 2. LLM이 역할 고정 모드를 무시하고 다른 역할 영웅을 추천하는 문제

**증상**: 딜러로 플레이 중인데 "힐러가 케어를 안 해줘요"라고 하면 힐러 교체를 추천했다.

**원인**: LLM이 맥락상 자연스러운 해결책(힐러를 바꿔라)을 제안하는 경향이 있었다.

**해결**: 역할 고정 규칙을 프롬프트 최상단에 "절대 규칙"으로 명시하고 

답변에 포함된 영웅 이름을 파싱해 허용 목록 밖의 영웅을 사후에 치환하는 이중 방어 로직을 추가했다. 

다만 사용자가 메시지에서 직접 언급한 영웅 이름(같은 편 동료를 가리키는 경우 등)은 치환 대상에서 제외한다.

<br><br>

## 3. LLM이 조합 질문을 "내 영웅 교체" 질문으로 오인하는 문제

**증상**: "상대가 바스티온, 토르비욘으로 압박하는데 팀 조합을 어떻게 짤까?"를 

LLM이 `intent=swap`으로 분류해 본인 영웅(예: 디바)을 교체해야 한다는 답변을 했다.

**원인**: 영웅 이름이 여러 개 등장하면 LLM이 `swap` 의도로 오판하는 경향이 있었다.

**해결**: SWAP INTENT GUARD를 도입했다. 

`intent=swap`으로 분류됐는데 메시지 원문에 `current_hero` 이름이 직접 등장하지 않으면 

`intent=general`로 재분류하고 `swap_guard_triggered` 플래그를 설정한다. 

이 플래그가 있을 때만 `current_hero_uncertain=True`로 처리해 역할 제한 없이 자유로운 조합 답변이 나오게 했다.

<br><br>

## 4. LLM이 힐 불만을 "힐러 플레이 중"으로 오해하는 문제

**증상**: "힐을 못 받아서 자꾸 죽는다"고 하면, LLM이 `current_hero_role=support`로 분류해 힐러 교체를 추천했다.

**원인**: "힐"이라는 단어가 들어있으면 LLM이 힐러를 플레이 중이라고 추론하는 패턴이 있었다.

**해결**: "힐" 키워드와 부정/부족 표현이 근접해서 등장하면 힐 수급 불만으로 재분류하는 정규식을 추가했다.

```py
HEAL_COMPLAINT_PATTERN = re.compile(r"힐[^.!?\n]{0,6}(못|안|부족|없|끊)")
if llm_hero_role == "support" and HEAL_COMPLAINT_PATTERN.search(message):
    llm_hero_role = context.get("current_hero_role")  # 실제 역할로 교정
```

화이트리스트(특정 문구를 일일이 나열)로는 "힐을 못줘", "힐이 끊겨" 같은 변형을 전부 잡기 어렵기 때문에 패턴 매칭으로 더 넓게 커버했다.

<br><br>

## 5. 후속 질문에서 이전 대화 맥락이 갑자기 사라지는 문제

**증상**: "모이라 스탯 분석해줘" 후 "케어 우선순위 알려줘"라고 하면 모이라와 전혀 무관한 범용 답변이 나왔다.

**원인**: `intent=general` + 영웅 이름 없음 → `current_hero_uncertain=True`로 처리되어 세션 컨텍스트가 통째로 무시됐다.

**해결**: `current_hero_uncertain` 판단 기준을 `swap_guard_triggered` 플래그로 한정했다. 

단순히 `intent=general`인 것만으로는 불확실로 판단하지 않는다. 

SWAP INTENT GUARD가 실제로 발동한 경우(조합 질문을 교체 질문으로 오인한 경우)에만 

`current_hero_uncertain=True`가 설정되도록 제한해 정상적인 후속 질문의 맥락이 끊기는 문제를 해결했다.
