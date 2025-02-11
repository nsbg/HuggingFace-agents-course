## 🤖 에이전트
### 정의
- 사용자가 정의한 목표를 달성하기 위해 AI 모델을 활용하여 환경과 상호 작용하는 시스템

- **추론**, **계획**, **작업 실행**을 결합하여 작업 수행
    - 작업 실행을 위해 외부 도구를 사용하는 경우도 있음

### 구성 요소
- 두뇌(AI 모델): 추론, 계획을 처리하며 상황에 따라 어떤 액션을 취할지도 결정

- 신체(툴): 에이전트가 수행할 수 있는 모든 기능
    - ⚠️ 행동 != 툴 ⚠️

    - 특정 행동을 수행하기 위해 여러 개의 도구를 결합할 수 있음

### 특징
- 에이전트를 위한 AI 모델은 일반적으로 입출력 형태가 모두 텍스트인 LLM이 많이 사용됨

- 툴 설계는 에이전트 품질에 직결되기 때문에 매우 중요함

## 🔠 LLM
- Context length

    - LLM이 처리할 수 있는 최대 토큰 수와 최대 어텐션 범위

## ✉️ 메시지, 스페셜 토큰
### 메시지
- 시스템 메시지(=시스템 프롬프트)

    - 모델의 작동 방식 정의 → 모든 상호 작용에 적용되는 지침 역할

    - 예시
        ```python
        system_message = {
            "role": "system",
            "content": "You are a professional customer service agent. Always be polite, clear, and helpful."
        }
        ```

- 대화

    - 인간(사용자)과 LLM(어시스턴트)이 주고 받는 메시지로 구성됨

    - 대화 템플릿을 이용해 이전 대화 내용을 저장하고 컨텍스트를 유지하여 일관성 있는 멀티턴 대화 가능

    - 예시
        ```python
        conversation = [
            {"role": "user", "content": "I need help with my order"},
            {"role": "assistant", "content": "I'd be happy to help. Could you provide your order number?"},
            {"role": "user", "content": "It's ORDER-123"},
        ]
        ```
    
    - SmolLM2 대화 템플릿을 적용한 대화 예시
        ```
        <|im_start|>system
        You are a helpful AI assistant named SmolLM, trained by Hugging Face<|im_end|>
        <|im_start|>user
        I need help with my order<|im_end|>
        <|im_start|>assistant
        I'd be happy to help. Could you provide your order number?<|im_end|>
        <|im_start|>user
        It's ORDER-123<|im_end|>
        <|im_start|>assistant
        ```

### 대화 템플릿
- 유저와 어시스턴트 사이 대화를 구조화하여 모델마다 다른 스페셜 토큰에도 불구하고 각 모델이 적합한 형식의 프롬프트를 수신할 수 있도록 함

- **기본 모델 vs 인스트럭트 모델**

    - 기본 모델: '다음 토큰 예측'을 위해 대규모의 원시 데이터로 학습됨 - `SmolLM2-135M`

    - 인스트럭트 모델: 지시사항을 따르고 대화에 참여할 수 있도록 파인튜닝된 모델 - `SmolLM2-135M-Instruct`

- 기본 모델을 인스트럭트 모델처럼 작동하게 하려면 모델이 이해할 수 있는 일관된 방식으로 프롬프트를 구성해야 함

    - 이 때 대화 템플릿이 매우 중요함

- 기본 모델은 다양한 대화 템플릿으로 파인튜닝될 수 있기 때문에 인스트럭트 모델을 사용할 때는 올바른 템플릿인지 확인해야 함

## 🛠️ 도구
- LLM에 주어진 기능

- 달성하고자 하는 명확한 목표가 있음

    - Web search, 이미지 생성, retrieval 등

- 도구에 포함되어야 하는 것

    - 함수가 수행하는 작업에 대한 텍스트 설명

    - 호출 가능한 요소
    - 매개변수와 그에 대한 타입 정의
    - (선택사항) 출력 결과와 출력 결과에 대한 타입 정의

- 작동 방식

    - 에이전트에서 LLM은 텍스트를 입력으로 받아 텍스트를 출력하는 것까지만 처리 가능
    
    - LLM은 스스로 툴을 호출할 수 없기 때문에 필요할 때 LLM이 툴을 호출하는 텍스트를 생성하도록 요청

    - 에이전트는 LLM의 출력을 파싱하여 툴 호출 후 툴의 출력 결과는 다시 LLM으로 전달되어 최종 응답이 구성됨

- LLM과 도구 연결 방식

    - **시스템 프롬프트**를 사용해 툴에 대한 설명을 LLM에 제공

    - 정확한 구조를 갖는 JSON 형식으로 제공되지만 꼭 JSON 형식이 아니더라도 정확하고 일관된 형식이라면 어떤 것도 가능

### 서식 지정 자동화 툴
- 툴 구현에서 중요한 것은 이름, 기능, 예상 입력 형태, 출력 형태

    - 예시

        ```python
        # Python 데코레이터 사용

        @tool
        def calculator(a: int, b: int) -> int:
            """Multiply two integers."""
            return a * b

        print(calculator.to_string())
        ```

## ⚙️ AI 에이전트 작동 흐름
### 핵심 요소
- thinking(Thought) → acting(Act), observing(Observe)

    - **Thought**: 에이전트 내 LLM에서 다음 단계를 결정

    - **Action**: 에이전트가 목적에 맞는 툴을 호출하여 작업 수행

    - **Observation**: 툴의 출력 결과가 LLM의 답변에 반영됨

### Thought-Action-Observation 사이클
- 에이전트가 목표를 달성할 때까지 사이클이 반복됨

- 많은 에이전트 프레임워크에서는 모든 사이클이 정의된 로직을 따를 수 있도록 규칙, 가이드라인을 시스템 프롬프트에 포함시킴

### 예시: 날씨 에이전트
- 사용자 입력

    - "뉴욕의 현재 날씨가 어때?"

- Thought-Action-Observation 사이클 수행 과정

    - 첫 번째 사이클

        - Thought: 내부 추론 → "사용자가 뉴욕의 현재 날씨를 알고 싶어 하니까 날씨 데이터를 가져올 수 있는 도구에 접근해야겠군!"

        - Action: 툴 사용 → 날씨 API를 호출할 수 있는 JSON 형식의 명령 수행

        - Observation: 툴을 호출한 후 얻은 결과 값을 프롬프트에 추가
    
    - 두 번째 사이클

        - Updated thought: observation을 바탕으로 내부 추론 결과 업데이트

        - Final action: updated thought를 활용해 사용자에게 최종 응답 제공
    
## 🧠 Thought: 내부 추론, Re-Act 접근 방식
### 개요
- Thought는 주어진 태스크를 해결하기 위해 에이전트의 내부에서 수행되는 추론과 계획 단계를 의미함

- 현재 관찰된 내용을 통해 다음 단계를 결정

- 예시

    - Planning, Analysis, Decision Making, Problem Solving, Memory Integration, ...

- 참고

    - ❓ 함수 호출을 위해 파인튜닝된 LLM에서 thought 과정은 선택 사항

### Re-Act
- **Re**asoning+**Act**ing

    - "Let's think step by step"과 같은 표현을 프롬프트에 포함시키는 기법

        - ❓ CoT가 Re-Act 방식 중 하나인건가?

## 🏃 Action: 에이전트가 환경과 상호 작용할 수 있도록 지원
### 개요
- Action은 AI 에이전트가 환경과 상호 작용하기 위해 취하는 구체적인 단계에 해당

### 에이전트의 종류
- 에이전트 유형

    |유형|설명|
    |:--:|--|
    |JSON Agent|JSON 형식으로 수행해야 할 작업 지정|
    |Code Agent|에이전트가 외부에서 해석되는 코드 블럭 작성|
    |Function-calling Agent|각 작업에 대해 메시지를 생성할 수 있도록 파인튜닝된 JSON Agent의 하위 카테고리|

- 액션이 끝나면 더이상 새로운 토큰을 생성하지 않는 것이 에이전트의 중요한 역할 중 하나

    - 모든 형식의 에이전트에 해당됨

        - 이 과정을 통해 의도치 않은 출력을 방지하고 사용자에게 정확한 응답을 제공하도록 보장

### Stop and Parse
- 에이전트의 출력이 구조화되고 예측 가능하도록 보장하기 위해 사용되는 방식

    - 구조화된 형식으로 결과 출력: 에이전트가 의도한 작업을 미리 정해둔 형식에 맞게 출력

    - 추가 토큰 생성 중지: 작업이 끝나면 새로운 토큰 생성 중지

    - 출력 구문 분석: 외부 파서가 정해진 형식의 작업을 읽고, 호출해야 하는 툴을 결정하고, 필요한 매개변수 추출

### Code Agents
- 단순히 JSON 객체만을 출력하는 것을 넘어 코드 에이전트가 실행 가능한 코드 블럭을 생성

- 장점

    - 표현력: 반복문, 조건문, 중첩된 함수 같이 복잡한 로직 표현 가능

    - 모듈화 및 재사용성: 다양한 태스크와 액션에 재사용 가능

    - 쉬운 통합: 외부 라이브러리나 API와 직접 통합할 수 있음

- 예시

    ```python
    # 코드 에이전트 예제: 날씨 정보 가져오기
    def get_weather(city):
        import requests

        api_url = f"https://api.weather.com/v1/location/{city}?apiKey=YOUR_API_KEY"
        
        response = requests.get(api_url)
        
        if response.status_code == 200:
            data = response.json()
            return data.get("weather", "No weather information available")
        else:
            return "Error: Unable to fetch weather data."

    # 함수 실행 및 최종 답변 출력
    result = get_weather("New York")
    final_answer = f"The current weather in New York is: {result}"
    
    print(final_answer)
    ```

- 코드 에이전트는 Stop and Parse를 따름
    
    - 생성한 코드 블럭은 끝나는 시점이 정해져 있음 → `Stop`

    - 코드 블럭 실행 결과를 활용해 원하는 데이터를 추출하거나 로직 수행 → `Parse`

## 👀 Observe: 피드백 반영 및 적용
### 개요
- 에이전트가 Action의 결과를 인지하는 단계

- Observation 단계에서 에이전트의 역할

    - 피드백 수집: 액션의 성공 또는 실패에 대한 데이터 수집

    - 결과 추가: 기존 컨텍스트에 새로운 정보를 추가하여 메모리 업데이트

    - 전략 수정: 업데이트된 컨텍스트를 사용해 더욱 정교한 thought와 action 수행

- Observation의 종류

    |유형|예시|
    |:--:|--|
    |System Feedback|에러 메시지, 상태 코드|
    |Data Changes|DB 업데이트, 파일 시스템 수정, 상태 변화화|
    |Environmental Data|센서 값 읽기, 자원 사용량|
    |Response Analysis|API 응답, 쿼리 결과, 연산 결과|
    |Time-based Events|예약된 태스크 완료|