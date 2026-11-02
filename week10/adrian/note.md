
# ERD

Regex 문법으로 표현해 봤음.

- user+ - org*
- (user|org) - repo*
- (user|org) - payment_plan?
- repo - fork*
- repo - (issue|pr)*
- (issue|pr) - comment*
- comment - reaction*
