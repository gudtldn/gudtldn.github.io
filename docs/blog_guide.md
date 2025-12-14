# Jekyll Front Matter 속성 가이드

Jekyll의 포스트 파일(`_posts/` 폴더 내의 `.md` 또는 `.markdown` 파일)에는 YAML 프론트 매터(front matter)를 사용하여 다양한 메타데이터와 설정을 지정할 수 있다.

> post를 작성할 때, 파일명은 반드시 `YYYY-MM-DD-제목.md` 형식으로 작성되어야 한다.

---

## 필수 및 자주 사용되는 속성

| 속성명           | 설명                                             | 예시                                        |
| ---------------- | ------------------------------------------------ | ------------------------------------------- |
| layout           | 사용할 레이아웃 지정 (`_layouts` 폴더 내 파일명) | `layout: post`                              |
| title            | 글 제목                                          | `title: "C++26 컴파일 타임 리플렉션"`       |
| description      | 글 설명 (SEO 및 SNS 공유용)                      | `description: "C++26 리플렉션 소개"`        |
| image            | 대표 이미지 경로 (SEO, SNS 공유용)               | `image: /assets/images/cpp26-reflect.png`   |
| date             | 작성 날짜 (파일명과 다를 경우 우선 적용)         | `date: 2025-06-20`                          |
| author           | 작성자 이름 또는 닉네임                          | `author: 홍길동`                            |
| categories       | 카테고리(들), 문자열 또는 리스트로 지정          | `categories: [C++]`                         |
| tags             | 태그(들), 문자열 또는 리스트로 지정              | `tags: [c++, c++26, reflection]`            |
| permalink        | 이 포스트의 고유 URL 경로 지정                   | `permalink: /blog/cpp-reflection/`          |
| published        | 게시 여부 (false면 빌드 시 미출력)               | `published: false`                          |
| excerpt          | 요약문 (일부 테마/SEO에서 사용)                  | `excerpt: "이 글은 C++26 리플렉션 소개..."` |
| last_modified_at | 마지막 수정일                                    | `last_modified_at: 2025-06-21`              |
| toc              | 목차 표시 여부 (테마에 따라 다름)                | `toc: true`                                 |
| toc_sticky       | 목차 고정 여부 (테마에 따라 다름)                | `toc_sticky: true`                          |
| math             | 수식 렌더링 여부 (MathJax 등 사용 시)            | `math: true`                                |
| mermaid          | Mermaid 다이어그램 렌더링 여부                   | `mermaid: true`                             |

[예시](https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/_posts/2019-08-08-text-and-typography.md?plain=1)
