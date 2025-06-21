## 🎯 프로젝트 소개

이 블로그는 **앞으로의 모든 모각코** 학습 동안의 여정을 체계적으로 기록하기 위해 만들어졌습니다. 기존 알고리즘 중심 블로그와는 별도로, 다양한 기술 스택 학습과 프로젝트 개발 과정을 문서화합니다.

기존에는 알고리즘 풀이 중심의 블로그만 운영했지만, 모각코에서는 더 폭넓은 활동이 이루어지므로 별도의 전용 공간이 필요하다고 판단하여 개설하게 되었습니다.

## 🛠 기술 스택

### 정적 사이트 생성

- **Hugo**: Go 기반 정적 사이트 생성기
- **Hugo Theme Stack**: 깔끔하고 현대적인 블로그 테마
- **Markdown**: 콘텐츠 작성 마크업


### 배포 및 호스팅

- **GitHub Pages**: 정적 사이트 호스팅
- **GitHub Actions**: 자동 빌드 및 배포 파이프라인
- **Custom Domain**: 선택적 도메인 연결 지원


### 개발 도구

- **Git**: 버전 관리
- **VS Code**: Markdown All in One extension 활용
- **Hugo CLI**: 로컬 개발 서버


## 🚀 시작하기

### 로컬 개발 환경 설정

1. **저장소 클론**
```bash
git clone https://github.com/your-username/your-repo-name.git
cd your-repo-name
```

2. **Hugo 설치** (macOS)
```bash
brew install hugo
```

3. **로컬 서버 실행**
```bash
hugo serve -D
```

4. **브라우저에서 확인**
```
http://localhost:1313
```

## ⚙️ 배포 과정

### GitHub Actions 자동 배포

1. **커밋 \& 푸시**
```bash
git add .
git commit -m "Add new post: 제목"
git push origin main
```

2. **자동 빌드**: GitHub Actions가 Hugo 빌드 실행
3. **배포**: `gh-pages` 브랜치에 정적 파일 배포
4. **공개**: GitHub Pages를 통해 자동 배포

### 배포 워크플로우

```yaml
# .github/workflows/hugo.yml
name: Deploy Hugo site to Pages
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
      - name: Build
        run: hugo --minify
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
```


## 📝 글쓰기 가이드

### Front Matter 예시

```yaml
---
title: "게시물 제목"
description: "게시물 설명"
slug: "url-friendly-slug"
date: 2025-06-21T16:05:00+09:00
author: "황현석"
categories: ["카테고리"]
tags: ["태그1", "태그2"]
weight: 1
---
```


### 마크다운 작성 팁

- **VS Code Markdown All in One**: 실시간 미리보기
- **이미지**: `static/img/` 폴더에 저장 후 `/img/filename.jpg`로 참조
- **코드 블록**: 언어별 syntax highlighting 지원
- **수식**: LaTeX 문법 지원 (`$...$`, `$...$`)


## 🎨 커스터마이징

### 테마 설정

- `config.toml`에서 사이트 전체 설정 관리
- 색상, 폰트, 레이아웃 등 Stack 테마 옵션 활용


### 카테고리별 스타일링

```yaml
# content/categories/개발/_index.md
---
title: 개발
description: 개발 관련 포스팅
style:
    background: "#e63946"
    color: "#fff"
---
```

## 📄 라이선스

이 프로젝트는 MIT 라이선스 하에 배포됩니다. 자세한 내용은 [LICENSE](LICENSE) 파일을 참조하세요.

## 🤝 기여하기

개인 학습 블로그이지만, 오타나 개선사항이 있다면 이슈를 통해 알려주세요!

---
