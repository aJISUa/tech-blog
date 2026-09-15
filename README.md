# aJISUa Tech Blog

수업, 동아리, 프로젝트에서 배우고 만든 것들을 정리하는 Jekyll 블로그입니다.

- 테마: [Minimal Mistakes](https://github.com/mmistakes/minimal-mistakes)
- 배포: GitHub Pages
- 주소: <https://ajisua.github.io/tech-blog/>

## 글 쓰기

`templates/post.md`를 복사해서 `_posts` 아래에 넣습니다.

```text
_posts/<활동-폴더>/YYYY-MM-DD-<글-이름>.md
```

글 맨 위 설정에서 두 가지만 정하면 홈과 기록 페이지에 자동으로 정리됩니다.

| 항목 | 뜻 | 예시 |
| --- | --- | --- |
| `categories` | 큰 분류 | `강의`, `동아리`, `프로젝트`, `스터디`, `대외활동` |
| `series` | 어떤 활동에 속한 글인지. 같은 이름끼리 묶임 | `JAVA프로그래밍및실습II`, `캡스톤디자인과창업프로젝트` |

```yaml
---
title: "[GDGoC] Git 브랜치 전략 스터디"
date: 2026-10-02 20:00:00 +0900
series: "GDGoC Ewha"
categories:
  - 동아리
tags:
  - Git
excerpt: "동아리 스터디에서 브랜치 전략을 정리했다."
---
```

- `series`가 없는 글은 해당 분류의 "그 밖의 글"로 모입니다.
- 분류가 보이는 순서는 `_data/record_categories.yml`에서 바꿀 수 있습니다. 여기에 없는 분류는 맨 뒤에 붙습니다.
- 이미지는 `assets/images/<폴더>/`에 넣고 `{{ '/assets/images/<폴더>/<파일>' | relative_url }}`로 씁니다.

## 주요 경로

| 경로 | 용도 |
| --- | --- |
| `_posts/` | 글 |
| `templates/post.md` | 글 템플릿 |
| `_pages/records.html` | 분류, 활동별 글 모아보기 (`/records/`) |
| `_includes/record-categories.html` | 분류 순서를 계산하는 조각 |
| `_data/record_categories.yml` | 분류 순서 |
| `_data/navigation.yml` | 상단 메뉴 |
| `_config.yml` | 블로그 이름, 주소, 테마 설정 |
| `assets/css/main.scss` | 블로그 디자인 |
| `on-care.html` | On-Care 개발기 전체 페이지 (`/on-care/`) |

예전 강의 페이지 주소 `/courses/`는 `/records/`로 자동 이동합니다.

## 로컬 실행

GitHub Pages가 쓰는 Jekyll 3.9는 Ruby 3.2 이상과 맞지 않아서 Ruby 3.1에서 실행합니다.

```bash
bundle install
bundle exec jekyll serve
```

Docker를 쓴다면 저장소 폴더에서 다음처럼 실행할 수 있습니다.

```bash
docker run --rm -p 4000:4000 -v "$PWD:/srv/blog" -w /srv/blog ruby:3.1 bash -c "bundle install && bundle exec jekyll serve --host 0.0.0.0"
```

브라우저에서 <http://localhost:4000/tech-blog/>를 엽니다.

## 배포

`main` 브랜치에 올리면 `.github/workflows/pages.yml`이 빌드해서 GitHub Pages에 배포합니다. 저장소의 **Settings → Pages → Build and deployment → Source**는 **GitHub Actions**로 되어 있어야 합니다.
