# 사이드 프로젝트 작업 구조

여러 아이디어를 목업으로 찍어보고, 마음에 드는 것만 발전시키고, 나머지는 기록으로 남기기 위한 최소한의 관리 구조.

## 진짜 소스는 Notion, GitHub는 배포만

**"작업실 허브" > "프로젝트 목록" Notion 데이터베이스**가 프로젝트 목록/상태/아이디어 백로그의 단일 소스다. 상태를 바꾸거나 새 아이디어를 적는 건 전부 Notion에서 한다(폰에서도 바로 가능).

`registry.js`는 그 Notion DB의 **스냅샷**이다 — 직접 손으로 고치지 않는다. Notion에서 내용을 바꾼 뒤 "노션 동기화해줘"라고 요청하면, Claude가 Notion을 읽어서 `registry.js`를 다시 생성하고 커밋한다. 브라우저(정적 페이지)가 Notion API를 직접 호출하지 않는 이유는 API 키가 클라이언트 코드에 노출되기 때문 — 그래서 "동기화 → 정적 파일 커밋 → push" 흐름을 쓴다.

## 상태(status) 4단계 (Notion "상태" select와 동일)

| 상태 | 의미 |
|---|---|
| 아이디어 | 아직 목업도 없음, repo 없음. 허브에는 카드가 아니라 짧은 메모로만 표시 |
| 프로토타입 | 목업을 만들어서 써보는 중. 계속할지 아직 결정 전 |
| 발전중 | 마음에 들어서 계속 키우는 중 (기능 추가, 실사용, 배포 준비 등) |
| 히스토리 | 써봤지만 계속 안 하기로 함. 저장소는 지우지 않고 그대로 두고(라이브 URL도 안 죽음), 허브에는 접힌 기록으로만 표시 |

## 저장소 구조: 프로젝트마다 독립 GitHub 저장소

허브와 각 프로젝트는 **서로 다른 GitHub 저장소**로 각각 배포된다. GitHub Pages 기준으로:

| 저장소 | 라이브 주소 | 로컬 폴더 |
|---|---|---|
| `tossneon.github.io` | `https://tossneon.github.io/` | `index.html`, `registry.js` (이 폴더 루트) |
| `checknote` | `https://tossneon.github.io/checknote/` | `체크노트_실사용/` |
| `dividend-passbook` | `https://tossneon.github.io/dividend-passbook/` | `초간단 배당현황/` |
| `baby-place-registry` | `https://tossneon.github.io/baby-place-registry/` | `아기랑 갈곳/` |
| `kpc-coach-chat` | `https://tossneon.github.io/kpc-coach-chat/` | `KPC 코칭챗봇/` |

저장소 이름이 정확히 `tossneon.github.io`인 것 하나만 계정의 루트 주소로 서빙되고, 나머지는 저장소명이 그대로 하위 경로가 되는 GitHub Pages 기본 규칙이다. 이 로컬 폴더(`claude/`)는 편집 작업 공간이고, 각 프로젝트 폴더 안에 **자기만의 `.git`**이 따로 있다(허브 저장소는 그 폴더들을 추적하지 않고 `.gitignore`로 제외한다).

## `registry.js` (Notion에서 생성되는 스냅샷)

```js
window.REGISTRY = {
  pagesUrl: repo => `https://tossneon.github.io/${repo}/`,
  statuses: [...],
  projects: [ { id, title, repo, date, status, tags, description, note }, ... ]  // status:"아이디어"는 repo 없이도 됨
};
```

## 새 아이디어가 생기면

1. **아직 목업 안 만들 거면** → Notion "프로젝트 목록"에 상태="아이디어"로 행 하나 추가. 그게 끝. (다음 동기화 때 허브의 "아이디어 메모"에 자동으로 뜸)
2. **목업을 만들기로 하면**:
   - 폴더 하나 새로 생성 (`{프로젝트명}/`), 그 안에 `index.html`(게시본) + `{slug}.jsx`(소스)를 넣는다.
   - 그 폴더 안에서 `git init` + 첫 커밋
   - `gh repo create {slug} --public --source=. --push` 로 저장소 생성 + push
   - 저장소 Settings → Pages에서 `main` 브랜치, 루트(`/`)로 배포 켜기 (한 번만 하면 이후 push마다 자동 반영)
   - Notion에서 그 항목의 상태를 "프로토타입"으로, repo 속성에 `{slug}` 입력

## 써보고 난 뒤

- **마음에 들면** → Notion에서 그 프로젝트의 상태를 "발전중"으로만 바꾼다. 저장소는 그대로 쓰고, 기능이 늘면 그 저장소에 계속 커밋 쌓으면 된다.
- **접기로 하면** → 상태를 "히스토리"로 바꾸기만 하면 된다. 저장소는 지우지 않는다 — 허브의 "히스토리" 섹션에 자동으로 접혀서 들어가고, 라이브 URL도 그대로 살아있다.
- 두 경우 다 "노션 동기화해줘"라고 해야 허브 화면에 실제 반영됨(동기화 → 커밋 → 승인 후 push).

## 파일 규칙

- 폴더명: 한글 프로젝트명 그대로 (`아기랑 갈곳`, `초간단 배당현황` 등) — 로컬 작업 공간용
- 저장소/URL slug: 영문 (`baby-place-registry`, `dividend-passbook` 등) — 폴더명과 문자 그대로 안 맞아도 됨
- 각 프로젝트 폴더 안: `index.html`(게시본, Pages가 이 파일을 서빙) + `{slug}.jsx`(React 소스, 참고용)

## 배포 워크플로우 (Notion → git → live)

1. Notion "프로젝트 목록"에서 상태/설명/아이디어 등 수정
2. "노션 동기화해줘" → Claude가 Notion을 읽어 `registry.js` 재생성
3. 프로젝트 코드 자체를 수정했다면 해당 프로젝트 폴더에서도 수정
4. **승인 후에만** 해당 저장소에 `git add` + `git commit` + `git push` — push되는 순간 GitHub Pages가 자동으로 다시 빌드해서 1분 내 반영
5. `registry.js`가 바뀌었으면 허브 저장소(`tossneon.github.io`)도 따로 커밋 + push해야 허브 화면에 반영됨

승인 없이 임의로 push하지 않는다. 로컬 커밋까지는 자유롭게 쌓아도 되지만, 원격(`git push`)은 매번 명시적으로 확인받고 진행한다. 새 저장소를 만드는 것도 마찬가지로 매번 진행 전에 확인한다.

## 왜 이렇게 했는가

기존에는 허브 파일이 두 벌(`maker-journal.jsx`, `maker-journal-hero.jsx`) 있었고 프로젝트 데이터를 양쪽에 손으로 따로 적다 보니 벌써 어긋나 있었다. `registry.js` 하나로 합쳐서 문제를 해결했지만, 그마저도 손으로 JS 파일을 고치는 건 번거롭고 폰에서는 사실상 불가능했다. Notion을 진짜 소스로 두면 폰에서도 상태 하나 드래그로 바꿀 수 있고, 정적 파일(`registry.js`)은 그 스냅샷 역할만 하니 "고칠 곳은 항상 한 곳"이라는 원칙은 그대로 유지된다. 저장소를 프로젝트별로 나눈 건 이미 `checknote`가 독립 저장소로 배포되어 있던 방식을 그대로 따른 것 — 프로젝트마다 커밋/배포 이력이 섞이지 않고, 하나가 잘못돼도 다른 프로젝트에 영향이 없다.
