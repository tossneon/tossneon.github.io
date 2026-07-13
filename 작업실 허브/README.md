# 작업실 허브 — 소스 아카이브

실제로 배포되는 허브 페이지는 여기가 아니라 저장소 루트의 [`index.html`](../index.html)이다 (정적 호스팅은 보통 루트의 `index.html`을 홈페이지로 서빙하기 때문).

이 폴더에는 React 버전 소스만 참고용으로 남겨둔다:

- `maker-journal-hero.jsx` — 현재 `index.html`의 기반이 된 디자인
- `maker-journal.jsx` — 이전 카드형 버전 (참고용, 더 이상 사용 안 함)

허브의 데이터(프로젝트 목록/상태/아이디어)는 이 폴더가 아니라 [`../registry.js`](../registry.js) 한 곳에서만 관리한다.
