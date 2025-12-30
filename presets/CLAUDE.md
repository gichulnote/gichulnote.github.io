# 프리셋 데이터

## 목적
기출노트 앱에서 사용하는 시험별 정답 프리셋

## 디렉토리 구조
```
gichulnote/
├── presets/                        # 프리셋 (public 배포)
│   ├── index.json                  # 프리셋 목록
│   ├── CLAUDE.md
│   └── {preset-id}/
│       ├── meta.json
│       └── answers/{session-id}.json
├── data/                           # 원본 PDF (private)
│   └── {시험명}/{연도}/
│       ├── {월}-questions.pdf
│       └── {월}-answers.pdf
└── scripts/                        # 유틸리티 스크립트
    └── pdf_to_thumbs.sh
```

## 프리셋 추가 작업 절차

### 1. 데이터 준비
```bash
# data/{시험명}/{연도}/ 에 PDF 배치
# - {월}-questions.pdf (문제지)
# - {월}-answers.pdf (정답지)
```

### 2. 썸네일 생성 (페이지 매핑용)
```bash
mkdir -p data/{시험명}/{연도}/thumbs/{월}
pdftoppm -png -r 75 data/{시험명}/{연도}/{월}-questions.pdf data/{시험명}/{연도}/thumbs/{월}/page
```

### 3. 정답 이미지 추출
```bash
pdftoppm -png -r 150 data/{시험명}/{연도}/{월}-answers.pdf data/{시험명}/{연도}/temp/answer-{연도}-{월}
```

### 4. JSON 파일 생성
**중요: 모든 페이지를 직접 확인하여 문제-페이지 매핑 작성 (추정 금지)**

```json
{
  "sessionId": "2024-01",
  "label": "2024년 1월",
  "answers": [1, 2, 3, ...],  // 전체 문제 정답 배열
  "questionPageMap": {
    "1": 1, "2": 1, "3": 1,   // "문제번호": 페이지번호
    "4": 2, "5": 2,
    ...
  }
}
```

### 5. meta.json 업데이트
```json
{
  "id": "jaegyeong",
  "name": "재경관리사",
  "questionCount": 120,
  "choicesCount": 4,
  "timeLimit": 150,
  "sections": [
    { "name": "재무회계", "from": 1, "to": 40, "totalScore": 100, "passingScore": 70 }
  ],
  "sessions": [
    { "id": "2024-01", "label": "2024년 1월" }
  ]
}
```
- sessions는 최신순 정렬

### 6. 커밋 & 푸시
```bash
git add .
git commit -m "feat: {시험명} {연도} 프리셋 추가"
git push
```

## 시험별 정보

### 재경관리사 (jaegyeong)
- 문제 수: 120문제 (4지선다)
- 시간: 150분
- 섹션: 재무회계(1-40), 세무회계(41-80), 원가관리회계(81-120)
- 과락: 섹션별 70점

### 공인중개사 (realtor)
- TODO: 정보 추가 필요

## 주의사항
1. **페이지 매핑은 반드시 모든 페이지 확인** - 문제 길이가 페이지마다 다름
2. 정답 이미지에서 정답 추출 시 섹션별로 구분해서 확인
3. meta.json의 sessions 배열은 최신 회차가 먼저 오도록 정렬
4. **여러 회차 처리 시 병렬 에이전트 사용** - 각 회차는 독립 작업이므로 Task 도구로 동시 처리 (직접 순차 처리 금지)
