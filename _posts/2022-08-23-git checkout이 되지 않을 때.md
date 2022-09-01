---
categories: Git
tags: [git, error]
---
    
## git checkout이 되지 않을 때 
git checkout이 되지 않는다.

Please commit your changes or stash them before you switch branches. 에러가 뜬다.

```
# 현재 Staging 영역에 있는 파일의 변경사항을 스택에 넣어둔다. 
$ git stash

# stash 명령어로 스택에 넣어둔 변경 사항을 적용하고, 스택에서 제거해준다.
$ git stash pop
```

[[Git] Git Merge 또는 Git checkout 오류 해결하기](https://blog.hodory.dev/2020/02/18/error-Your-local-changes-would-be-overwritten-by-merge/)
