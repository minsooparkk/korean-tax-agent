# TaxDoctor 세무 검토 (taxdoctor-tax-review)

한국 세법 질문에 **조문·판례·해석례 원문을 근거로 붙여** 검토의견을 쓰는 Claude 스킬입니다.
공개 TaxDoctor 세법 DB(MCP)를 **읽기 전용**으로 조회하며, **무료이고 로그인·API 키가 없습니다.**
답변은 **실무 참고용이며 세무 자문이 아닙니다.**

[taxdoctorai.com](https://taxdoctorai.com)의 세무 검토 파이프라인(질문 분석 → 근거 검색 →
근거 검증 → 답변) 방법론을 스킬로 옮긴 공개 버전입니다. 플러그인에는 실행 코드가 없고,
Markdown 지침과 원격 MCP 서버 설정만 들어 있습니다.

```
> /tax 프리랜서에게 3.3% 원천징수했는데 매일 출근시키면 근로소득으로 재분류되나요?
```

**English summary** — A Claude skill for Korean tax questions. It looks up the current text of
Korean tax statutes, enforcement decrees, court and tax-tribunal decisions, National Tax Service
rulings, annexes, addenda and tax treaties through the public, read-only TaxDoctor MCP server
(`https://mcp.taxdoctorai.com/mcp`, free, no login or API key), then writes a review opinion in
Korean in which every legal statement carries a citation, and checks those citations against the
database before answering. It has an `easy` mode (search, answer, citation check) and a `hard` mode
(question analysis, evidence search, evidence check, answer). The plugin contains no executable
code. The output is reference material for practitioners, not tax or legal advice.

## 무엇을 하나

- **기억이 아니라 원문으로 답합니다** — 조문 번호·세율·한도를 기억으로 쓰지 않고, 답하기 전에
  현행 시행본 원문을 조회합니다. 조문마다 시행일이 붙습니다
- **조문에서 근거까지** — 출발 조문에서 위임 시행령·시행규칙, 그 조문을 인용한 판례·심판례·
  해석례, 별표, 부칙 적용례까지 관계를 따라 모읍니다
- **적용시점** — 경정청구·지난 과세기간 질문은 그날 시행 중이던 조문과 부칙 적용례·경과조치로
  판단하고, 내년 이후는 시행예정 개정을 확인합니다. 그 시점 조문을 확인하지 못하면 현행
  조문으로 대신 답하지 않고 "확인 불가"로 밝힙니다
- **조세조약** — 108개 체결국·지역의 조약 조문(국문·영문)과 그 조문을 고친 개정 의정서
- **세법 밖 전제 조문** — 민법·주택법·민간임대주택법·도시정비법·노동·사회보험 법령 등은
  "세법 아님" 표시와 함께 따로 조회합니다
- **인용 검증** — 답변에 적힌 조문·판례·문서번호가 DB에 실제로 있는지 마지막에 대조해, 없는
  번호·옛 조문 번호·틀린 조문 제목을 "인용 확인 메모"로 알립니다
- **환각 방지 설계** — 확인한 근거만 인용, 근거 강도에 맞춘 조건부 결론, 반대 근거 병기

검토가 대신해 주지 않는 것: 실제 신고·처분 대응의 판단, 사실관계 확인, 공식 유권해석.
결론은 반드시 원문과 세무 전문가의 확인을 거치세요.

## 설치

### Claude 디렉터리에서 추가 (claude.ai·Claude 데스크톱·Cowork)

claude.ai에서 **Customize › Plugins › Discover**를 열고 "TaxDoctor 세무 검토"를 찾아
추가합니다. 플러그인이 켜지면 MCP 서버가 함께 연결되고 `tax` 스킬을 쓸 수 있습니다.
(디렉터리 등재 전에는 아래 Claude Code 방법을 쓰세요.)

### Claude Code 마켓플레이스

Claude Code 안에서:

```
/plugin marketplace add kernelspaces/korean-tax-agent
/plugin install taxdoctor-tax-review@korean-tax-agent
```

### 스킬만 수동 설치

```bash
git clone https://github.com/kernelspaces/korean-tax-agent.git
cp -r korean-tax-agent/skills/tax ~/.claude/skills/
claude mcp add --transport http korea-tax-law https://mcp.taxdoctorai.com/mcp
```

claude.ai 커넥터나 다른 이름으로 `https://mcp.taxdoctorai.com/mcp`를 이미 연결해 두었다면
그대로 동작합니다. 스킬은 서버 이름이 아니라 도구 이름(`search_tax`, `get_article` …)으로 찾습니다.

## 사용법

```
/tax [easy|hard] <세무 질문>
```

| | **easy** (기본) | **hard** |
|---|---|---|
| 흐름 | 검색 → 답변 → 인용 검증 | 질문 분석 → 근거 검색 → 근거 검증 → 답변 → 인용 검증 |
| 소요 시간 | 2~5분 | 10분 이상 |
| 적합한 질문 | 기한·세율·요건 확인, 일반 설명 | 복합 쟁점, 해석 대립, 고액 리스크, 지난 과세기간 판단 |

- 모드를 생략하면 easy로 동작하고, 질문이 복합 쟁점이면 답변 말미에 hard 모드를 권합니다.
- 판단에 필요한 사실(취득일·거래 구조 등)이 빠지면 갈래별로 조건부로 답하고 말미에 묻습니다.
- 스킬을 부르지 않고 세무 질문을 해도 스킬 설명이 맞으면 자동으로 쓰입니다.
- 단계별 방법론은 [`skills/tax/references/`](skills/tax/references/)에 전부 공개돼 있습니다.

## 데이터 처리 — 무엇이 어디로 가나

이 플러그인은 로컬에서 아무것도 실행하지 않습니다(스크립트·훅·실행 파일·설치 패키지 없음).
파일을 읽거나 쓰지 않고, 자격 증명이나 환경 변수를 읽지 않으며, 대화 내용을 저장하지 않습니다.
하는 일은 Claude가 아래 원격 MCP 서버 하나(`.mcp.json`에 선언)의 도구를 부르게 하는 것뿐입니다.

| 서버 | 운영 | 보내는 것 | 받는 것 |
|---|---|---|---|
| `korea-tax-law` — `https://mcp.taxdoctorai.com/mcp` | TaxDoctor | Claude가 질문에서 뽑은 검색어·법령명·조문 번호·문서번호·기준일, 질문의 쟁점 문장(해석례 검색), 그리고 인용 검증 단계에서 **작성한 답변 초안 본문** | 조문·판례·결정례·해석례·별표·부칙·조약 원문 |

- 서버는 **읽기 전용**이고 로그인이 없어 이름·이메일 같은 계정 정보를 보내지 않습니다.
- `korea-tax-law` 서버는 검색어와 대화 내용을 기록하지 않고, 과도한 호출 차단과 장애 대응을
  위해 접속 IP·연결 프로그램 정보·호출한 도구 이름·처리 시간을 90일간 보관합니다. 실무 용어로
  조문을 찾을 때(`search_articles`)는 관련도 순위를 매기려고 검색어와 후보 조문 일부를 순위 제공자
  (TypeSafe AI, 미국)에 보냅니다. 자세한 내용: [TaxDoctor 개인정보처리방침](https://taxdoctorai.com/privacy).
- TaxDoctor DB에 없는 법령(참조 법령 목록에도 없는 세법 밖 법령·조례)이나 수록 범위 앞 시점의
  조문은 다른 곳에서 찾지 않습니다. 답변이 그 대목을 "확인하지 못했다"고 밝히고 국가법령정보센터
  (law.go.kr) 확인을 안내합니다.
- 질문에 주민등록번호·계좌번호처럼 검토에 필요 없는 개인정보는 넣지 마세요. Claude가 그 내용을
  검색어로 옮길 수 있습니다.

## 데이터 소스

**korea-tax-law — TaxDoctor 세법 MCP** ([연결 도움말](https://taxdoctorai.com/mcp-guide))

국세청 조세법령 목록의 현행 국세·지방세 법률·시행령·시행규칙(90개)과 지난 시점 판, 부칙의
적용례·경과조치, 법원 판례·조세심판원 결정례·국세청 심사·이의·과세전적부심사 결정례,
국세청·법제처·행정안전부 해석례, 별표·서식, 시행예정 개정, 조세조약을 본문까지 담은 그래프
DB입니다. 세법과 따로 표시된 비세법 참조 법령과 조정대상지역 지정 이력도 있습니다. 데이터는
국가법령정보센터 등 공공 원천에서 갱신됩니다. 도구 20개(`search_tax`, `get_evidence`,
`search_articles`, `get_article`, `get_article_tiers`, `get_article_history`, `get_amendments`,
`list_upcoming`, `search_cases`, `search_interpretations`, `search_annexes`, `get_annex`,
`verify_citations`, `list_laws`, `list_treaties`, `search_treaties`, `get_treaty_article`,
`search_reference_articles`, `get_reference_article`, `get_regulated_area_history`)가 모두 읽기 전용입니다.

## 구조

```
korean-tax-agent/
├── .claude-plugin/
│   ├── plugin.json           # 플러그인 매니페스트
│   ├── icon.png              # 목록 아이콘 (TaxDoctor)
│   └── marketplace.json      # Claude Code 마켓플레이스 정의
├── .mcp.json                 # 원격 MCP 서버 하나 (플러그인을 켜면 연결)
└── skills/
    └── tax/
        ├── SKILL.md          # 오케스트레이터 — 모드 라우팅, 실행 규칙, 인용 검증
        └── references/       # 단계별 방법론 (hard 모드 각 단계의 지침)
            ├── query-analysis.md
            ├── source-search.md
            ├── fact-check.md
            └── answer-generation.md
```

## 버전

- **v2.1.0** — Claude 디렉터리 접수용 정리. 지금 MCP 서버의 도구 20개 스키마에 맞춤(약칭+조번호
  즉답·AI 관련도 순위, `get_article` 기준일 판, `get_amendments`, `get_article_tiers`, 비세법 참조
  법령, 조정대상지역 이력). 원본 파이프라인 지침의 최근 규칙 반영(사실관계·방향 고정, 기한 질문의
  기산일 조문, 조특법 공통 제한 조문, 계산은 등호 산식으로, 사용자 정정·반박 다루기, "확인 불가"는
  적용 부정이 아님). README에 데이터 처리 안내 추가. 플러그인 이름을 `taxdoctor-tax-review`로 바꾸고
  저장소를 kernelspaces 조직으로 옮김. 보조 MCP(korean-law) 연결을 뺌 — TaxDoctor DB의 비세법 참조
  법령 계층이 그 역할을 맡고, 없는 법령은 "확인하지 못했다"로 밝힘. 목록 아이콘 추가
- **v2.0.0** — TaxDoctor 세법 MCP 중심으로 전환. 통합 그래프 탐색, 별표·시행예정 개정·과거 시점
  조문, 답변 인용 검증 추가
- **v1.0.0** — 최초 공개. korean-law MCP 기반

## 문의

연결이 안 되거나 결과가 이상하면 **hello@kernelspace.io**로 알려 주세요. 쓰신 질문과 받은 답을
함께 보내 주시면 빨리 확인할 수 있습니다.

TaxDoctor 를 쓴 사례와 질문은 재무 · 회계 · 세무 실무자 1,200명 넘게 모인 카카오톡 [재무 AI 오픈채팅방](https://open.kakao.com/o/gAUK185h)에서도 나눠요. TaxDoctor MCP 를 Claude 에 연결해 쓰는 영상은 유튜브 [AI CFO Lab](https://www.youtube.com/@AICFOLab)에 있어요.

## 면책

이 스킬의 답변은 AI가 공개 법령정보를 검색해 작성한 참고 의견이며, 세무대리인의 검토를
대체하지 않습니다. 실제 신고·처분 대응 전 반드시 회계사 또는 세무사와 상담하세요. 답변의
정확성·완전성은 보장되지 않으며, 이를 근거로 한 의사결정의 책임은 사용자에게 있습니다.

## 라이선스

[MIT](LICENSE)
