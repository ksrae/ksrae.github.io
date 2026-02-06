---
title: "How to avoid git push security"
date: 2025-01-16 12:46:00 +0900
comments: true
categories: javascript
tags: [github]
---


Have you ever encountered a situation where your code push was blocked by GitHub Push Protection during the `git push` process? This feature is a security system by GitHub designed to prevent sensitive security information (e.g., Access Keys, Tokens, etc.) from being pushed to the repository.

In this post, I'll share my experience with a blocked push and provide a step-by-step guide to how I resolved the issue.

# Problem Statement

During the development testing of a Slackbot, I directly added the Slack API Token to the code. When I tried to push the commit containing this token, the following error message appeared:

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

Even a force push (`push --force`) didn't solve the problem.

# Root Cause Analysis

GitHub Push Protection automatically detects and blocks commits containing sensitive security information such as private keys and tokens.

In my case, the cause was the code containing the Slack API Token, with the relevant commit details as follows:

```
commit: 8292d3c0f2414a62f1455cb65db363094c9631f2
path: chatbot.js:4
```

# Resolution Steps

To resolve the push block, you need to remove the sensitive security information that triggered the protection. Generally, the following steps should be followed:

## Removal of Sensitive Information

Sensitive information should be separated into an environment variable file (.env), and that file should be added to .gitignore.

```
SLACK_API_TOKEN=xoxb-xxxxxx...
```

### Modifying the Commit Containing Sensitive Information

Use the `git rebase` or `git filter-repo` commands to remove the sensitive information from the commit.

Example:

```
git rebase -i <problematic_commit_id>^
```

Or

```
git filter-repo --path chatbot.js --invert-paths
```

However, in my case, the problem persisted even after completing the `.gitignore` settings. Therefore, I chose the bypass method provided by GitHub through the provided link.

## Utilizing the Unblock Secret Link

The following link is provided at the end of the error message:

```
https://github.com/(USER_NAME)/(GIT_TITLE)/security/secret-scanning/unblock-secret/(UNBLOCK_KEY)
```

### Accessing the Link

Open the link in a browser to navigate to the GitHub's Unblock Secret page.

### Selecting a Safe Reason

Select a reason why it is safe to include the key in the repository or write a relevant explanation.

### Allowing Push

After submitting the input, try pushing again.

```
git push origin master
```

The push should proceed normally.

# Important Considerations

- **Environment Variable Management:** Be careful not to include sensitive information directly in the code. Using environment variable files and adding them to `.gitignore` is a common practice.
- **Git Commit Management:** Sensitive information included in past commits can cause push blocks. Develop a habit of managing your commit history.
- **GitHub Documentation Reference:** Consult the GitHub Push Protection documentation for further details.

# Reference

[Working with push protection from the command line](https://docs.github.com/ko/code-security/secret-scanning/working-with-secret-scanning-and-push-protection/working-with-push-protection-from-the-command-line#resolving-a-blocked-push)

## Link
[한국어(Korean) Page](https://velog.io/@ksrae/git-push%ED%95%A0-%EB%95%8C-security%EB%A1%9C-%EC%9D%B8%ED%95%9C-%EC%97%90%EB%9F%AC-%EC%9A%B0%ED%9A%8C)
<br/>