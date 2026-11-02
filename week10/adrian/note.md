
# ERD

Regex 문법으로 표현해 봤음.

- user+ - org*
- (user|org) - repo*
- (user|org) - payment_plan?
- repo - fork*
- repo - (issue|pr)*
- (issue|pr) - comment*
- comment - reaction*
- user - star*
- repo - star*

# Access Pattern

- user/org 화면: repo 목록 / payment plan
- user 화면: star 누른 repo 목록 / 가입한 org 목록
- repo 화면: issue/pr 목록(w/ open 수), fork 목록(w/ 개수), 새 issue/pr 작성 기능, fork 생성 기능, star 기능(w/ 개수)
- issue/pr 화면: 코멘트 목록, 각 코멘트 반응 개수와 반응자 id들
