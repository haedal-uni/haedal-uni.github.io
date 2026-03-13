---
categories: Git
tags: [git, error]
---

# git reset --hard가 되지 않을 때
특정 커밋으로 `git reset --hard` 하려 했지만 

이전 커밋에 특수문자(`:`)가 포함된 파일명이 있어 오류가 발생했다. (Windows 환경)

Could not reset index file to revision ~

<br>

아래와 같이 해결했다.


```bash
# 백업 
git branch backup

# working tree에 영향x
git branch temp-reset < commit 해시>

# 원격 main을 해당 커밋으로 강제 덮어쓰기
git push origin +temp-reset:main
```

