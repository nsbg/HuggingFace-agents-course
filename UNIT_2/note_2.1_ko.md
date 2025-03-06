## 🖼️ smolagents
### 장점
- **단순성**: 프레임워크를 쉽게 이해하고 적용하며 확장할 수 있도록 코드 복잡성과 추상화 최소화

- **유연한 LLM 지원**: Hugging Face 툴 및 외부 API와의 통합을 통해 다양한 LLM과 호환 가능

- **코드 우선 접근**: CodeAgents가 직접 코드를 작성하도록 지원하기 때문에 파싱 과정이 없어도 되며 툴 호출을 단순화함

- **HF Hub와의 통합**

### Smolagents 사용이 적합한 경우
- **가볍고 간단한** 솔루션이 필요할 때

- 복잡한 환경설정 없이 **빠른 실험**을 진행하고 싶을 때

- 애플리케이션 로직이 **단순하고 명확**할 때

## 🖥️ CodeAgents로 에이전트 구축하기
### CodeAgents 특징
- 전통적인 Multi-step 에이전트 프로세스는 툴을 정하기 위해 JSON 형식을 사용함

- LLM이 툴을 정하기 위해 JSON 파일을 파싱하는 것보다 코드를 직접 사용하는 게 툴 호출에 더 효과적이라는 연구 결과가 있음

- 코드를 사용한 액션 수행 장점

    - **조합성**: 다양한 액션을 쉽게 결합하고 재사용 가능

    - **객체 관리**: 복잡한 구조(예: 이미지)를 직접 다룰 수 있음

    - **일반성**: 계산 가능한 태스크라면 모두 표현할 수 있음

    - **LLM이 학습한 방식과 잘 맞음**: 고품질의 코드 데이터가 LLM 학습 데이터에 포함되어 있음

### Code Agent 작동 방식
- `CodeAgent.run()` 작동 방식

    ![codeagent_diagram](./img/codeagent_diagram.png)

- `CodeAgent`는 기존 변수와 지식을 에이전트의 문맥에 통합하고 단계를 거쳐 액션 수행

    - 1. 시스템 프롬프트는 `SystemPromptStep`에 저장되고 사용자 쿼리는 `TaskStep`에 기록됨

    - 2. 액션이 수행될 때까지 아래 과정 반복

        - 2-1. `agent.write_memory_to_messages` 메서드는 에이전트의 로그를 LLM이 처리할 수 있는 대화 메시지 목록에 기록

        - 2-2. 변환된 메시지는 모델로 전송되고 모델은 이에 대한 응답 생성

        - 2-3. 생성된 응답으로부터 실행할 액션(CodeAgent에서는 이 액션이 코드 스니펫이어야 함) 파싱

        - 2-4. 파싱된 코드 실행

        - 2-5. 실행 결과는 메모리 내부의 `ActionStep`에 기록됨

- Agent 내부에서 import 사용하기

    - `smolagents`는 Python 코드 스니펫을 작성하고 실행하는 에이전트를 전문적으로 다루기 때문에 보안을 위해 샌드박스 환경에서 실행 제공
    
    - 코드 실행에는 엄격한 보안 조치를 적용되어 기본적으로 미리 정의된 안전 목록에 없는 모듈은 import가 차단됨
    
    - `additional_authorized_imports`에 문자열로 추가적인 import를 지정하면 해당 모듈은 import 가능

## 🧩 코드 스니펫 또는 JSON blob으로 액션 작성하기
### Tool Calling Agents
- `smolagents`에서 제공하는 두 번째 유형의 에이전트

- Python 코드 스니펫을 사용하는 `CodeAgent`와 다르게 JSON 형식으로 툴 호출 생성

    - OpenAI, Anthropic을 포함해 많은 곳에서 사용하는 표준 방식

- ToolCallingAgent vs CodeAgent

    - ToolCallingAgent
        ```json
        [
            {"name": "web_search", "arguments": "Best catering services in Gotham City"},
            {"name": "web_search", "arguments": "Party theme ideas for superheroes"}
        ]
        ```
    
    - CodeAgent
        ```Python
        for query in [
            "Best catering services in Gotham City", 
            "Party theme ideas for superheroes"
        ]:
            print(web_search(f"Search for: {query}"))
        ```

- 복잡한 툴 호출이 필요하지 않은 간단한 시스템에서는 `ToolCallingAgent`가 효과적으로 쓰일 수 있음

    - `smolagents`는 전반적으로 성능이 더 뛰어난 `CodeAgents`에 포커스를 맞춤

### Tool Calling Agent 작동 방식
- Code Agent의 다단계 워크플로우를 따름

- Code Agent와의 차이점 → **⭐작업 구성 방식⭐**

    - Tool Calling Agent는 툴 이름과 파라미터를 지정하는 JSON 객체 생성

        - 모델은 생성된 JSON 객체를 해석하여 적절한 툴 실행

## 🛠️ 툴
### 개요
- `smolagents`에서 툴은 LLM이 에이전트 시스템 내에서 호출할 수 있는 함수처럼 쓰임

- 툴과 상호작용하기 위해 LLM이 필요로 하는 주요 컴포넌트
    
    - **이름**

    - **툴 설명**: 툴 역할 설명

    - **Input 타입&설명**: 툴에 전달되는 인자에 대한 설명

    - **Output 타입**: 툴의 반환값

- 앞에서 다룬 `AlfredAgent`에서 쓰인 웹 검색 툴 예시

    - **이름**: `web_search`

    - **툴 설명**: 특정 쿼리에 대한 웹 검색 수행

    - **Input**: `query` (string) - 검색 내용

    - **Output**: 검색 결과를 포함하고 있는 string

### `smolagents` 툴 생성 방법
- 간단한 함수 기반 툴을 위한 **`@tool` 데코레이터** 사용

    - `smolagents`는 내부적으로 Python 코드 내 함수의 기본 정보 자동 분석

        - 명확한 함수를 작성하면 LLM이 해당 함수를 더욱 쉽게 활용할 수 있음

    - 함수 정의 시 고려해야 할 요소

        - **명확한 함수명**

            - LLM이 함수의 목적을 잘 이해할 수 있도록 함

        - **입출력에 대한 타입 힌트**

            - 함수가 제대로 쓰일 수 있도록 입출력 값의 타입을 명확하게 지정해야 함
        
        - **`Args:` 섹션을 활용한 자세한 설명**

            - 각 인자의 의미에 대한 설명은 LLM에게 중요한 맥락을 제공하기 때문에 신중하게 작성해야 함수

- 복잡한 함수를 위한 **`Tool` 클래스의 하위 클래스** 생성

    - 함수 대신 클래스를 사용하면 **메타데이터**를 추가하여 LLM이 해당 툴을 더 효과적으로 활용할 수 있도록 함

    - 클래스에서 정의해야 할 요소

        - `name`

        - `description`: 에이전트의 시스템 프롬프트에 포함될 설명

        - `inputs`: `type`과 `description`을 키로 가지는 딕셔너리

        - `output_type`: 반환될 출력값의 타입 명시

        - `forward`: 툴의 실제 실행 로직을 포함하는 메서드
    
    - 예시
        ```Python
        from smolagents import Tool, CodeAgent, HfApiModel

        class SuperheroPartyThemeTool(Tool):
            name = "superhero_party_theme_generator"
            description = """
            This tool suggests creative superhero-themed party ideas based on a category.
            It returns a unique party theme idea."""

            inputs = {
                "category": {
                    "type": "string",
                    "description": "The type of superhero party (e.g., 'classic heroes', 'villain masquerade', 'futuristic Gotham').",    
                }
            }

            output_type = "string"

            def forward(self, category: str):
                themes = {
                    "classic heroes": "Justice League Gala: Guests come dressed as their favorite DC heroes with themed cocktails like 'The Kryptonite Punch'.",
                    "villain masquerade": "Gotham Rogues' Ball: A mysterious masquerade where guests dress as classic Batman villains.",
                    "futuristic Gotham": "Neo-Gotham Night: A cyberpunk-style party inspired by Batman Beyond, with neon decorations and futuristic gadgets."
                }
                
                return themes.get(category.lower(), "Themed party idea not found. Try 'classic heroes', 'villain masquerade', or 'futuristic Gotham'.")

        # 툴 초기화
        party_theme_tool = SuperheroPartyThemeTool()

        agent = CodeAgent(tools=[party_theme_tool], model=HfApiModel())

        # 에이전트 실행
        result = agent.run(
            "What would be a good superhero party idea for a 'villain masquerade' theme?"
        )

        print(result)
        ```

### `smolagents` 기본 제공 툴
- **PythonInterpreterTool**

- **FinalAnswerTool**

- **UserInputTool**

- **DuckDuckGoSearchTool**

- **GoogleSearchTool**

- **VisitWebpageTool**

### 툴 공유 및 import
- 공유

    ```bash
    tool_name.push_to_hub("{your_username}/party_theme_tool", token="<YOUR_HUGGINGFACEHUB_API_TOKEN>")
    ```

- import

    ```Python
    from smolagents import load_tool

    image_generation_tool = load_tool(
        "HF_USER_ID/TOOL_NAME",
        trust_remote_code=True
    )
    ```

- Hugging Face Space import

    - Hugging Face Space를 Tool로 import 할 수 있음
    
        ```Python
        from smolagents import Tool

        image_generation_tool = Tool.from_space(
            "black-forest-labs/FLUX.1-schnell",
            name="image_generator",
            description="Generate an image from a prompt"
        )
        ```
    
    - Gradio 백엔드와 연결되기 때문에 `gradio_backend` 설치 필요

- LangChain Tool import

    ```Python
    from langchain.agents import load_tools

    search_tool = Tool.from_langchain(load_tools(["serpapi"])[0])
    ```

## 📜 Agentic RAG system 구축하기
- Agentic RAG

    - 기존 RAG 시스템에 **자율적인 에이전트와 동적 지식 검색(dynamic knowledge retrieval)** 결합

        - ❓ 동적 지식 검색의 정의

    - 기존 RAG 시스템의 한계

        - 단일 검색 단계에 의존적

            - LLM은 한 번의 검색 결과만 가지고 답변 생성

            - 추가적인 검색이 필요할 경우 이를 반영하지 못함
        
        - 사용자 입력과 직접적으로 의미적 유사성을 가지는 내용에만 초점을 맞춤

            - 검색된 데이터가 사용자 입력 내용과 직접적으로 일치하지 않으면 중요한 정보가 누락될 가능성이 있음 → ❓ 직접적으로 일치한다는 것의 범위
    
    - 검색과 생성 프로세스를 컨트롤하여 더 정확한 응답을 제공할 수 있음

        - 기존 RAG 방식에서 LLM은 검색된 데이터를 기반으로 답변
    
    - Agentic RAG를 활용해 기존 RAG 방식의 한계 극복 가능

        - 에이전트가 자율적으로 검색 쿼리 생성

        - 에이전트가 검색 결과의 신뢰도를 분석하고 필요할 경우 추가 검색 진행

        - 다중 검색 단계 수행

## 🤝 멀티 에이전트
- 다양한 에이전트를 결합하여 복잡한 태스크를 처리할 수 있게 함

    - 단일 에이전트에 의존하지 않고 서로 다른 기능을 가진 에이전트로 태스크를 분산

- 일반적인 멀티 에이전트 구성

    - Manager 에이전트: 전체 작업 관리 및 다른 에이전트로 적절한 하위 작업 배분

    - Code interpreter 에이전트: Python 코드 생성 및 실행

    - Web search 에이전트: 웹에서 정보를 검색(retrieval)하고 관련 데이터 제공