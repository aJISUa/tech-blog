# aJISUa Tech Blog

강의별 학습 기록과 프로젝트 개발 경험을 일자별로 정리하는 Jekyll 기술 블로그입니다.

- 테마: [Minimal Mistakes](https://github.com/mmistakes/minimal-mistakes)
- 배포: GitHub Pages
- 블로그 주소: <https://ajisua.github.io/tech-blog/>

## 글 작성 방법

`templates/course-daily-post.md`를 복사해 `_posts` 아래에 저장합니다.

```text
_posts/<강의-슬러그>/YYYY-MM-DD-<글-슬러그>.md
```

예를 들어 Python 기초 강의 3일차 글은 다음처럼 저장할 수 있습니다.

```text
_posts/python-basic/2026-09-13-functions.md
```

글 상단의 Front Matter에서 `course` 값이 같은 글들은 **강의** 페이지에 자동으로 묶입니다.

```yaml
---
title: "함수와 매개변수"
date: 2026-09-13 20:00:00 +0900
course: "Python 기초"
day: 3
categories:
  - 강의
tags:
  - Python
  - 함수
excerpt: "Python 함수 선언과 매개변수 전달 방식을 정리합니다."
toc: true
---
```

강의가 다섯 개라면 각 글의 `course`에 다섯 강의명 중 하나를 적기만 하면 됩니다. 별도의 강의 페이지를 직접 만들 필요는 없습니다.

## 주요 경로

| 경로 | 용도 |
| --- | --- |
| `_posts/` | 공개되는 학습 기록과 프로젝트 글 |
| `templates/course-daily-post.md` | 일자별 강의 글 템플릿 |
| `_pages/courses.html` | 강의별 글 자동 모아보기 |
| `_data/navigation.yml` | 상단 메뉴 |
| `_config.yml` | 블로그 이름, 주소, 테마 설정 |
| `on-care.html` | 기존 On-Care 전체 개발 회고 |
| `assets/css/main.scss` | 테마 위에 적용하는 블로그 디자인 |

## 기존 On-Care 글

기존 단일 페이지는 삭제하지 않고 `/on-care/` 경로에 그대로 보존했습니다. 블로그 홈에는 이 글로 이동하는 프로젝트 글이 표시됩니다.

## 로컬 실행

Ruby와 Bundler가 설치된 환경에서 다음 명령을 실행합니다.

```bash
bundle install
bundle exec jekyll serve
```

브라우저에서 <http://localhost:4000/tech-blog/>를 엽니다.

## 배포

`main` 브랜치에 변경 사항을 올리면 `.github/workflows/pages.yml`이 사이트를 빌드하고 GitHub Pages에 배포합니다. 저장소의 **Settings → Pages → Build and deployment → Source**는 **GitHub Actions**로 설정해야 합니다.
