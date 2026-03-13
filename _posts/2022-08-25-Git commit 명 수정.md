---
categories: Git
tags: [git]
---
    
커밋명을 작성하다가 이슈번호를 자꾸 까먹고 enter를 치다보니 

커밋명을 수정하는 경우가 생겨서 기록했다.

```bash
git pull origin main
git commit --amend -m "new 커밋메세지"
git push -f origin main
```
*push -f는 강제로 push 하는 것이므로 주의한다.
