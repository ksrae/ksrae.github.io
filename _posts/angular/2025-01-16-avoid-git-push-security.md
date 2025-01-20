---
title: "git push할 때 security로 인한 에러 우회(How to avoid git push security)"
date: 2025-01-16 12:46:00 +0900
comments: true
categories: javascript
tags: [github]
---

개발 중 git push 과정에서 GitHub Push Protection으로 인해 코드 푸시가 차단된 경험이 있으신가요? 이 기능은 중요한 보안 정보(예: Access Key, Token 등)가 저장소에 푸시되는 것을 방지하기 위한 GitHub의 보안 시스템입니다.

이번 글에서는 제가 겪었던 푸시가 차단된 사례를 공유하며, 문제를 해결한 과정을 단계별로 정리하겠습니다.

# 문제 상황
Slackbot 개발 테스트 과정에서 코드에 Slack API Token을 직접 추가했는데, 이를 포함한 커밋을 푸시하려 하자 아래와 같은 에러 메시지가 나타났습니다.

```
remote: Resolving deltas: 100% (6/6), completed with 3 local objects.
remote: error: GH013: Repository rule violations found for refs/heads/master.
remote:
remote: - GITHUB PUSH PROTECTION
remote:   —————————————————————————————————————————
remote:     Resolve the following violations before pushing again
remote:
remote:     - Push cannot contain secrets
remote:
remote:
remote:      (?) Learn how to resolve a blocked push
remote:      https://docs.github.com/code-security/secret-scanning/working-with-secret-scanning-and-push-protection/working-with-push-protection-from-the-command-line#resolving-a-blocked-push   
remote:
remote:
remote:       —— Slack API Token ———————————————————————————————————
remote:        locations:
remote:          - commit: 8292d3c0f2414a62f1455cb65db363094c9631f2
remote:            path: chatbot.js:4
remote:
remote:        (?) To push, remove secret from commit(s) or follow this URL to allow the secret.
remote:        https://github.com/(USER_NAME)/(GIT_TITLE)/security/secret-scanning/unblock-secret/{UNBLOCK_KEY}
remote:
remote:
remote:
To https://github.com/(USER_NAME)/(GIT_TITLE).git
 ! [remote rejected] master -> master (push declined due to repository rule violations)
error: failed to push some refs to 'https://github.com/(USER_NAME)/(GIT_TITLE).git'
```

강제 푸시(push --force)도 문제를 해결하지 못했습니다.

# 문제의 원인
GitHub Push Protection은 푸시하려는 커밋에 비공개 키, 토큰 등 민감한 보안 정보가 포함되어 있을 경우 이를 자동으로 감지해 차단합니다.
제 경우, Slack API Token이 포함된 코드가 원인이었으며, 해당 커밋은 다음과 같이 기록되었습니다.

```
commit: 8292d3c0f2414a62f1455cb65db363094c9631f2
path: chatbot.js:4
```

# 해결 방법
## 원인 제거
푸시 차단을 해결하려면 문제가 되는 보안 정보를 제거해야 합니다.
일반적으로 다음과 같은 절차를 따라야 합니다.

### 코드에서 민감한 정보 제거
보안 정보는 환경 변수 파일(.env)로 분리하고, 해당 파일을 .gitignore에 추가합니다.

```
SLACK_API_TOKEN=xoxb-xxxxxx...
```

### 민감한 정보가 포함된 커밋 수정
git rebase 또는 git filter-repo 명령어를 사용하여 해당 커밋에서 민감한 정보를 제거합니다.

예시:
```
git rebase -i <problematic_commit_id>^
```

또는

```
git filter-repo --path chatbot.js --invert-paths
```

그러나 제 경우 이미 .gitignore 설정을 완료했음에도 불구하고 문제가 지속되었습니다.
그래서 GitHub 제공 링크를 통한 우회 방법을 선택했습니다.

## Unblock Secret 링크 활용
에러 메시지의 끝부분에는 다음과 같은 링크가 제공됩니다.

```
https://github.com/(USER_NAME)/(GIT_TITLE)/security/secret-scanning/unblock-secret/(UNBLOCK_KEY)
```

### 링크에 접속
브라우저에서 링크를 열어 GitHub의 Unblock Secret 페이지로 이동합니다.

### 안전한 이유 선택
해당 키를 저장소에 포함시켜도 안전한 이유를 선택하거나, 관련 설명을 작성합니다.

### Push 허용
입력 내용을 제출한 후 다시 푸시를 시도합니다.

```
git push origin master
```

푸시가 정상적으로 진행됩니다.

# 참고 사항
- 환경 변수 관리
민감한 정보를 코드에 직접 포함시키지 않도록 주의하세요.
환경 변수 파일을 사용하고 이를 .gitignore에 추가하는 것이 일반적인 방법입니다.

- Git 커밋 관리
과거 커밋에 포함된 민감한 정보가 푸시 차단의 원인이 될 수 있으므로, 커밋 히스토리를 관리하는 습관을 들이세요.

- GitHub 문서 참고
관련 문서: GitHub Push Protection

# Reference
[명령줄에서 푸시 보호 작업](https://docs.github.com/ko/code-security/secret-scanning/working-with-secret-scanning-and-push-protection/working-with-push-protection-from-the-command-line#resolving-a-blocked-push)