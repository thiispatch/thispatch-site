# ThisPatch 웹사이트

ThisPatch(런타임 공급망 보안 솔루션 Hechi) 소개 및 문의 접수용 정적 웹사이트입니다.

## 페이지
- `index.html` — 메인 (제품 소개 · 작동 방식 · FAQ · 문의 폼)
- `terms.html` — 이용약관
- `privacy.html` — 개인정보 수집·이용 동의
- `marketing.html` — 마케팅 목적 개인정보 수집·이용 동의

## 로컬에서 보기
```bash
python3 -m http.server 8000
# 브라우저에서 http://localhost:8000
```

## 배포
`main` 브랜치가 GitHub Pages로 자동 배포됩니다.

## 개발 워크플로
- `main`(배포) / `dev`(통합) 두 브랜치를 상시 유지합니다.
- 작업은 `dev`에서 분기한 기능 브랜치에서 진행하고, PR로 `dev`에 병합합니다.
- 배포는 `dev` → `main` PR로 릴리스합니다. (자세한 규칙은 팀 Git 가이드 참조)
