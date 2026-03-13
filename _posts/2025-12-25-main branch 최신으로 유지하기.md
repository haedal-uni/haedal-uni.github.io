---
categories: Git
tags: [git]
---

협업에서는 pull을 중요하게 다룬다.

pull 없이 push를 하면 내 변경 사항을 기준으로 히스토리가 재작성되면서

다른 사람이 먼저 main에 반영한 변경 사항이 충돌하거나 사라지는 상황이 발생한다.

<br>

보통 main에서 새로운 branch를 checkout해 작업을 진행하는데

그 사이 누군가 main에 PR을 merge했다면 해당 변경 사항을 반드시 내 브랜치에 반영해야 한다.

<br>

```bash
git pull origin main
git checkout new_branch
git rebase main
git push -f origin new_branch
```
