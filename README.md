LLM Prompt Caching 활용하기
1. 개요
프롬프트 정규화(Normalization)
사용자1: "맛있는 파스타 레시피 알려줘"
사용자2: "파스타 레시피 좀 알려주세요"
→ 이런 유사한 질문들에 대해 매번 LLM API를 호출하지 않고 캐시된 응답 활용

프롬프트 정규화는 다양한 형태로 입력된 질문들을 표준화된 형태로 변환하는 과정으로 다음과 같은 단계를 거침:

텍스트 전처리
불필요한 공백 제거
대소문자 통일
특수문자 처리
의미적 정규화
동의어 처리 (예: "알려줘", "가르쳐줘", "설명해줘" → 동일한 의미로 처리)
어순 정규화
불필요한 수식어 제거
토큰화 및 임베딩
정규화된 텍스트를 토큰으로 변환
의미적 유사성을 보존하는 벡터 공간으로 매핑
주요 사용 시나리오
FAQ 응답과 같은 반복적인 질문
데이터 전처리나 포맷팅 작업
자주 사용되는 번역이나 요약 작업
작동 프로세스

KV Caching과의 차이점
목적

KV Caching:

단일 프롬프트 내에서의 attention states 재사용
자동회귀(autoregressive) 토큰 생성 과정에서 이전에 생성된 토큰의 key-value attention states를 캐싱
각 새로운 토큰 생성마다 attention을 다시 계산하는 것을 방지

Prompt Caching:

여러 프롬프트 간의 attention states 재사용
자주 사용되는 텍스트 세그먼트(예: 시스템 메시지, 문서 등)의 attention states를 미리 계산하여 저장
동일한 텍스트 세그먼트가 다른 프롬프트에 나타날 때 재사용

image

2. API 활용 방법
ChatGPT
기본적으로 prompt caching이 적용되며, 캐시 사용 시 비용이 절반으로 감소합니다.

모델	캐시되지 않은 입력 토큰	캐시된 입력 토큰	출력 토큰
gpt-4o-2024-08-06	$2.50	$1.25	$10.00
GPT-4o fine-tuning	$3.75	$1.875	$15.00
gpt-4o-mini-2024-07-18	$0.15	$0.075	$0.60
GPT-4o mini fine-tuning	$0.30	$0.15	$1.20
o1-preview	$15.00	$7.50	$60.00
o1 mini	$3.00	$1.50	$12.00
특징:

1,024 토큰 이상의 프롬프트에 자동 적용
128 토큰 단위로 증가하는 캐시 적용
캐시 유지 기간: 5-10분 (최대 1시간)
Claude
헤더에 anthropic-beta: prompt-caching-2024-07-31를 추가하여 사용

curl https://api.anthropic.com/v1/messages \
     --header "x-api-key: $ANTHROPIC_API_KEY" \
     --header "anthropic-version: 2023-06-01" \
     --header "content-type: application/json" \
     --header "anthropic-beta: prompt-caching-2024-07-31" \
     --data \
'{
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1024,
    "system": [
        {
            "type": "text",
            "text": "넌 법률문서를 해석하는 AI assistant야."
        },
        {
            "type": "text",
            "text": "여기 복잡한 법률 계약의 전문이 있어: [엄청 긴 법률 계약 문서]",
            "cache_control": {"type": "ephemeral"}
        }
    ],
    "messages": [
        {
            "role": "user",
            "content": "이 계약의 주요 약관은?"
        }
    ]
}'
가격 정책
Model	Base Input Tokens	Cache Writes	Cache Hits	Output Tokens
Claude 3.5 Sonnet	$3 / MTok	$3.75 / MTok	$0.30 / MTok	$15 / MTok
Claude 3 Haiku	$0.25 / MTok	$0.30 / MTok	$0.03 / MTok	$1.25 / MTok
Claude 3 Opus	$15 / MTok	$18.75 / MTok	$1.50 / MTok	$75 / MTok
주요 특징:

Caching은 일반 입력보다 25% 비싸지만, 캐시 히트 시 10% 비용으로 사용 가능
TTL: 5분
최소 토큰:
Sonnet, Opus: 1024 token
Haiku: 2048 token (이하 토큰은 캐싱 미지원)
3. 로컬 모델 설정
참조: https://github.com/MachineLearningSystem/24MLSYS-prompt-cache.git

지원 아키텍처
Llama2
Falcon
MPT
설치 및 설정
기본 requirements 설치
pip install -r requirements.txt
벤치마크 evaluation용 추가 설치
cd ./dependency/bleurt
pip install .
디렉토리 구조
root/
├── demo.py
├── examples/
│   └── code_generation_game.xml
├── requirements.txt
└── promptcache/  # 라이브러리 디렉토리
실행 방법
# 캐시 활성화
python demo.py --enable_cache=True

# 캐시 비활성화
python demo.py --enable_cache=False
