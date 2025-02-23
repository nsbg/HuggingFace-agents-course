## 🗣️ Function-calling
### 정의
- LLM이 속한 환경 안에서 액션을 취할 수 있게 하는 방법

- 에이전트 tool vs function-calling

    - 공통점

        - 환경에 대한 작업을 수행할 수 있는 기능을 모델에 제공함

    - 차이점

        - Function-calling은 다른 에이전트 기법보다 프롬프트에 덜 의존적

        - 에이전트는 function-calling을 통해 tool을 쓸 수 있도록 파인튜닝됨

### 모델이 액션을 '학습'하는 방법

- 일반적인 에이전트의 작동 방식

    - **Think**: 목적을 달성하기 위해 어떤 액션을 취할지 결정

    - **Act**: 올바른 파라미터로 작업을 형식화하고 목적을 달성하면 생성 중지

    - **Observe**: 실행 결과 사용

    - API를 통해 이루어지는 모델과의 일반적인 대화 예시
        ```Python
        conversation = [
            {"role": "user", "content": "I need help with my order"},
            {"role": "assistant", "content": "I'd be happy to help. Could you provide your order number?"},
            {"role": "user", "content": "It's ORDER-123"},
        ]
        ```
    
    - ⭐ Function-calling은 위의 대화에서 action이나 observation에 새로운 역할을 부여함
        
        - "assistant"가 직접 답변하는 것이 아니라 "function_call" 내부의 "retrieve_payment_status" 호출

        - "role": "tool"이 추가되어 `retrieve_payment_status` 의 결과 반환
        ```Python
        conversation = [
            {
                "role": "user",
                "content": "What's the status of my transaction T1001?"
            },
            {
                "role": "assistant",
                "content": "",
                "function_call": {
                    "name": "retrieve_payment_status",
                    "arguments": "{\"transaction_id\": \"T1001\"}"
                }
            },
            {
                "role": "tool",
                "name": "retrieve_payment_status",
                "content": "{\"status\": \"Paid\"}"
            },
            {
                "role": "assistant",
                "content": "Your transaction T1001 has been successfully paid."
            }
        ]
        ```
    
    - Function-calling을 위한 스페셜 토큰

        - `[AVAILABLE_TOOLS]`: 사용할 수 있는 tool 리스트 시작을 알리는 토큰
        
        - `[/AVAILABLE_TOOLS]`: tool 리스트 끝을 알리는 토큰

        - `[TOOL_CALLS]`: tool 호출 실행

        - `[TOOL_RESULTS]`: tool 호출 결과를 받아옴

        - `[/TOOL_RESULTS]`: observation 종료(모델이 다시 응답을 생성할 수 있음)
    
    - Function-calling을 지원하지 않는 모델이어도 스페셜 토큰을 활용해 funcion-calling 기능을 추가할 수 있음

## 🔍 Function-calling을 위한 모델 파인튜닝
### funcion-calling을 위한 모델 학습
- 사전학습 모델(예: [google/gemma-2-2b](https://huggingface.co/google/gemma-2-2b)에서 시작하면 instruction following, chat, function-calling을 모두 학습시켜야 함

- 인스트럭션 튜닝 모델(예: [google/gemma-2-2b-it](https://huggingface.co/google/gemma-2-2b-it))를 사용하면 **학습시켜야 하는 정보의 양을 줄일 수 있음**

### LoRA (Low-Rank Adaptation of Large Language Models)
- Transformer 레이어 중 선형 레이어에 한 쌍의 랭크 분해 행렬을 추가하는 방식

- 학습이 진행되는 동안 어댑터의 가중치만 업데이트하고 기존 모델의 나머지 가중치는 업데이트하지 않음(=freeze)

    ![LoRA Inference](https://huggingface.co/datasets/agents-course/course-images/resolve/main/en/unit1/blog_multi-lora-serving_LoRA.gif)
