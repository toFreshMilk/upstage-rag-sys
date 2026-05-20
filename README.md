# 📚 Upstage Solar RAG Prototype

> Upstage Solar API를 활용한 **문서 기반 질의응답(RAG)** 프로토타입입니다.
> PDF·이미지·DOCX 문서를 올리면 Solar가 내용을 이해하고, 한국어로 친절하게 답변합니다.

---

## ✨ 이 프로젝트가 하는 일

직접 업로드한 문서를 "AI 비서가 읽고 기억한 다음, 그 내용을 토대로 대답"하게 해주는 웹앱입니다.
간단히 정리하면 다음 흐름으로 동작합니다.

```
📄 문서 업로드 → 🔍 Upstage Document Parse → ✂️ 텍스트 청킹
                                                    ↓
💬 채팅 ← 🤖 Solar Pro3 추론 ← 🎯 유사 문서 검색 ← 🧠 임베딩 저장
```

1. **문서 파싱** — Upstage의 `document-digitization` API로 PDF·이미지·DOCX에서 텍스트를 뽑아냅니다. OCR도 자동입니다.
2. **청킹(Chunking)** — 긴 텍스트를 1,500자 단위로 자르고, 문맥 보존을 위해 앞뒤 청크가 200자씩 겹치도록 합니다.
3. **임베딩 & 저장** — 각 조각을 `embedding-passage` 모델로 벡터화해서 로컬 파일(`data/vectors.json`)에 저장합니다.
4. **검색** — 사용자가 질문하면 `embedding-query` 모델로 질문을 벡터화한 뒤, 코사인 유사도가 높은 상위 5개 청크를 가져옵니다.
5. **답변 생성** — 검색된 문서를 컨텍스트로 묶어 `solar-pro3`(추론 강도 high) 모델에게 전달, 한국어로 답을 받아옵니다.

> 💡 **자유 대화 모드**도 함께 제공해요. 채팅창 우측 상단 토글을 켜면 문서 검색 없이 Solar와 일반 대화도 할 수 있습니다.

---

## 🧩 주요 기능

| 기능 | 설명 |
|------|------|
| 📤 **다중 문서 업로드** | PDF, PNG/JPG/TIFF/BMP/GIF 이미지, DOCX/DOC 지원. 한 번에 여러 파일 선택 가능 |
| 🧠 **로컬 벡터 스토어** | 별도 DB 없이 `data/vectors.json`에 임베딩 저장 — 가볍게 띄워 보기 좋음 |
| 🔀 **모드 전환** | "문서 기반" ↔ "자유 대화" 토글로 즉시 전환 |
| 📊 **실시간 통계 대시보드** | 전체 문서 수, 누적 사용 토큰, 평균 응답속도(ms) 표시 |
| ✍️ **마크다운 렌더링** | 답변은 GFM(GitHub Flavored Markdown)으로 표·코드블록까지 예쁘게 표시 |
| 🪵 **서버 로그 자동 수집** | `scripts/log-runner.js`가 `next` 출력을 `server.log`에 자동 기록 |
| ⏱️ **타임아웃 안전장치** | 클라이언트·서버 양쪽에 60초 타임아웃을 두어 행 거는 일 방지 |

---

## 🚀 빠른 시작

### 1. 사전 준비물

- **Node.js** `20.19.2` 이상
- **pnpm** `8.9.0` 이상 (`npm install -g pnpm`)
- **Upstage Console API Key** — [console.upstage.ai](https://console.upstage.ai) 에서 발급

### 2. 설치

```bash
git clone <your-repo-url>
cd upsrag
pnpm install
```

### 3. 환경변수 설정

프로젝트 루트에 `.env.local` 파일을 만들고 발급받은 키를 넣어주세요.

### 4. 개발 서버 실행

```bash
pnpm dev
```

브라우저에서 [http://localhost:3100](http://localhost:3100) 열기 — 끝!

> 포트가 3000이 아닌 **3100**인 점에 주의하세요. (`package.json`의 `dev` 스크립트 참고)

---

## 📜 자주 쓰는 스크립트

모든 스크립트는 `scripts/log-runner.js`를 거쳐 실행되어, 콘솔 출력이 `server.log`에 함께 누적됩니다.

| 명령어 | 동작 |
|--------|------|
| `pnpm dev` | 개발 서버 실행 (포트 3100, Hot Reload) |
| `pnpm build` | 프로덕션 빌드 |
| `pnpm start` | 빌드 결과로 서버 실행 (포트 3100) |
| `pnpm lint` | ESLint로 코드 점검 |

---

## 🛠️ 기술 스택

**프레임워크 & 언어**
- ⚛️ Next.js `15.1.0` (App Router) + React `19`
- 🟦 TypeScript `5.7`

**UI**
- 🎨 Tailwind CSS `3.4` + `@tailwindcss/typography`
- 🪶 lucide-react (아이콘)
- 📝 react-markdown + remark-gfm

**AI / RAG**
- 🌞 Upstage Solar API
  - `document-digitization` — 문서 파싱 + OCR
  - `embedding-passage` / `embedding-query` — 문서/쿼리 임베딩 (분리)
  - `solar-pro3` — Reasoning 모드 채팅 모델
- 📦 OpenAI SDK (`openai` v4) — Upstage가 OpenAI 호환 인터페이스를 제공하기 때문에 그대로 사용

---

## 🗂️ 디렉토리 한눈에 보기

```
upsrag/
├── src/
│   ├── app/
│   │   ├── page.tsx              # 메인 페이지 (대시보드 + 업로드 + 채팅 레이아웃)
│   │   ├── layout.tsx            # 루트 레이아웃
│   │   └── api/
│   │       ├── embed/route.ts    # 📥 문서 파싱·청킹·임베딩 API
│   │       ├── chat/route.ts     # 💬 검색 + Solar 호출 API
│   │       └── log/route.ts      # 🪵 클라이언트 에러 로그 수신
│   ├── components/
│   │   ├── Dashboard.tsx         # 통계 카드 3종(문서 수·토큰·응답속도)
│   │   ├── FileSection.tsx       # 다중 파일 업로드 UI
│   │   ├── ChatSection.tsx       # 채팅 UI + 모드 토글
│   │   └── ErrorLogger.tsx       # 전역 에러 캐치 → /api/log
│   └── lib/
│       ├── constants.ts          # RAG 파라미터 (TOP_K, CHUNK_SIZE 등)
│       ├── chunking.ts           # 자연스러운 경계 기반 청킹 알고리즘
│       ├── vectorStore.ts        # 로컬 JSON 벡터 저장소 + 코사인 유사도 검색
│       ├── types.ts              # 공용 타입 정의
│       ├── logger.ts             # 클라이언트 로거
│       └── server-logger.ts      # 서버 로거
├── scripts/
│   └── log-runner.js             # next 명령 래퍼 (stdout/stderr → server.log)
├── data/
│   └── vectors.json              # ← 임베딩 저장소 (실행 후 자동 생성)
├── server.log                    # 누적 서버 로그
└── package.json
```

---

## ⚙️ RAG 파라미터 튜닝

`src/lib/constants.ts`에서 검색·청킹 동작을 조절할 수 있습니다.

```ts
export const RAG_CONFIG = {
    TOP_K: 5,           // 검색 시 가져올 청크 개수
    CHUNK_SIZE: 1500,   // 청크 한 조각의 문자 수
    CHUNK_OVERLAP: 200, // 청크 간 겹침 문자 수 (문맥 손실 방지)
};
```

**파라미터별 권장 조정 방향**

- 📈 **답변이 단편적이라면** → `TOP_K`를 7~10으로 늘려보세요. (단, 토큰 비용 증가)
- 🧱 **긴 문맥의 연관성이 약하다면** → `CHUNK_OVERLAP`을 300~500으로 늘리기.
- 🔍 **세밀한 사실 검색이 중요하다면** → `CHUNK_SIZE`를 800~1000으로 줄여 정밀도 향상.

---

## 🧪 사용 시나리오 예시

1. **사내 매뉴얼 Q&A** — 두꺼운 매뉴얼 PDF를 올린 뒤 "환불 정책 요약해줘" 같은 질문.
2. **논문 읽기 도우미** — 논문 PDF + 이미지 도표 함께 업로드 → "3장 실험 결과 설명해줘".
3. **계약서 분석** — DOCX 계약서 → "위약금 조항만 뽑아 표로 정리해줘".
4. **OCR 활용** — 손글씨/스캔 이미지 → 자동 OCR 후 검색 가능.

---

## ⚠️ 알아두면 좋은 한계

- **벡터 스토어가 파일 기반**이라 동시 쓰기에 약하고, 문서가 많아지면 검색 속도가 선형으로 느려집니다. 프로덕션에서는 pgvector, Qdrant, Pinecone 등으로 교체를 권장합니다.
- **인증/세션 없음** — 누구나 업로드·검색이 가능한 단일 사용자 프로토타입입니다.
- **벡터 삭제 UI 없음** — 데이터 초기화는 `data/vectors.json` 파일을 직접 삭제하면 됩니다.
- **API Key 노출 위험** — 현재 `NEXT_PUBLIC_` 접두사로 노출되어 있어 빌드 산출물에 포함될 수 있습니다. 실 서비스 전 반드시 서버 전용 환경변수로 옮기세요.
- **응답 60초 제한** — `solar-pro3`의 reasoning_effort가 `high`라 더 오래 걸리는 질문에서 타임아웃이 날 수 있습니다.

---

## 🩺 문제 해결 (Troubleshooting)

| 증상 | 원인/해결 |
|------|----------|
| `API Key missing` 응답 | `.env.local`에 `NEXT_PUBLIC_UPSTAGE_API_KEY` 설정 후 서버 재시작 |
| 업로드 후 검색이 안 됨 | `data/vectors.json` 파일이 생성됐는지 확인. 권한 문제면 디렉토리 권한 확인 |
| 504 Gateway Timeout | 질문이 너무 복잡하거나 문서가 거대함 → `TOP_K`를 줄이거나 질문을 쪼개기 |
| 한글이 깨져 보임 | 터미널 인코딩 확인 (Windows는 `chcp 65001`로 UTF-8 설정) |
| 포트 충돌 | `package.json`의 `-p 3100`을 다른 포트로 변경 |

문제가 계속되면 `server.log`를 먼저 살펴보세요. 모든 요청·에러가 기록되어 있어 디버깅에 큰 도움이 됩니다.

---

## 🗺️ 앞으로 해보면 좋을 것들 (Roadmap 아이디어)

- [ ] 벡터 DB를 pgvector/Qdrant로 교체
- [ ] 업로드된 문서 목록 + 개별 삭제 UI
- [ ] 답변에 출처 문서 인용(citation) 표시
- [ ] 스트리밍 응답 (현재는 `stream: false`)
- [ ] 사용자 인증 + 워크스페이스 분리
- [ ] 임베딩 재사용 캐시 (같은 청크 재업로드 시 스킵)
- [ ] 평가용 RAG 정확도 벤치마크 스크립트

---

## 📎 참고 자료

- [Upstage Console](https://console.upstage.ai) — API 키 발급
- [Upstage Solar 모델 문서](https://developers.upstage.ai/docs/getting-started/models)
- [Upstage Document Parse 가이드](https://developers.upstage.ai/docs/apis/document-parse)
- [Next.js 15 App Router 문서](https://nextjs.org/docs/app)

---

## 📄 라이선스

이 프로젝트는 학습/프로토타이핑 목적으로 만들어졌습니다. 사용 시 Upstage API의 이용약관을 함께 확인해주세요.

---