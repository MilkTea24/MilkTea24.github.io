# GitHub Blog 글쓰기 가이드

## 준비사항
- `https://github.com/sigmadream/gitblog-templates` 클론
- `[id].github.io`를 저장소 이름으로 설정

## 메타데이터를 기반으로 한 사이트 정의
- `_config.yml`
    - 사이트 제목, 간단한 소개와 같은 블로그 사이트 전반에 필요한 내용
    - `name`, `description`, `email`, `github` 등과 같이 필요한 항목을 수정하여 저장소에 반영

## 글쓰기
- _posts에서 진행
- 각 post 위에 메타데이터가 있음

## 이미지 삽입
- 정적 파일은 위치가 중요
![기사 표지]({{site.baseurl}}/images/20231002/01.jpg)

## 표 삽입
- markdown table generator
- 복잡한 표는 만들 수 없음 -> 이미지로 제공하기

## disqus
- 내 사이트에 댓글 추가 가능
- _config-yaml에서 disqus의 사이트 short name 추가
- disqus에 advanced domain에서 자신의 github 주소 추가

## Google Analytics
- 구글 애널리틱스 검색
- 측정 시장 -> 계정 만들기 -> 속성 이름(자신의 깃허브 이름)
- 생성 후 측정 ID 복사해서 _config-yaml에 붙여넣기