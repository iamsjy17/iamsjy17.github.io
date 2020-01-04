---
layout: post
title: "[Git]상황별로 가져다 쓰는 Git 명령어"
date: 2018-08-18 15:00:00
author: Jewoo.Song
categories: Git
tags: git git명령어
---

매일 git 을 쓰면서도 `unstaged file` 만 `discard`하려면 어떻게 하더라..?  
`index`에 있는 파일만 `working tree`로 보내려면 어떻게 하더라..?

몇개의 명령어를 처보고, help 를 찾아보고 할 때가 많아서 상황별로 명령어를 정리해 두기로 했습니다.

## Repository 생성

#### Local Repository 생성

git Repository 로 사용할 Directory 로 이동 후 명령어 입력

```bash
git init
```

#### 원격 Repository 에서 가져오기

[주소]에 해당하는 원격 Repository 를 로컬로 복사해온다.

```bash
git clone [주소]
```

#### Git Config 설정

```bash
git config --global user.name "사용자 이름"
git config --global user.email "사용자 이메일"
```

#### 원격 Repository 에 올리기

원격 저장소의 [url]을 [name]으로 추가한다.

```bash
git remote add [name][url]
```

추가한 url 에 특정 branch 로 로컬 commit 내역을 push 한다.

```bash
git push [name][branch]
```

## 파일 원상 복구

#### 개별 파일 취소

1.  working tree 취소

    워킹트리의 수정된 파일을 index 에 있는 것으로 원복한다.

    ```bash
    git checkout -- [파일이름]
    ```

    워킹트리의 수정된 파일을 HEAD 에 있는 것으로 원복한다.

    ```bash
    git checkout HEAD --(생략가능) [파일명]
    ```

2.  index 추가 취소
    index 에 추가한 것을 취소한다.(unstage) 워킹트리의 변경내용은 보존된다.
    ```bash
    git reset -- [파일명]
    git reset HEAD [파일명]
    ```

#### 전체 파일 취소

1. working tree 취소
   _워킹트리의 모든_ 수정된 파일의 내용을 HEAD 로 원복한다.

   ```bash
    git checkout HEAD .
   ```

2. index 추가 취소(변경을 커밋하지 않았을 때)

   마지막 커밋이후의 _워킹트리와 index_ 의 수정사항을 모두 HEAD 로 되돌린다. (untracked 파일은 제외)

   ```bash
    git reset --hard HEAD
    git checkout -f
   ```

#### untracked 파일 제거

```bash
git clean -f
```

디렉토리까지 제거

```bash
git clean -f -d
```

#### 참조 : reset 옵션

- --soft : index 보존, 워킹트리 보존
- --mixed : index 취소, 워킹트리만 보존 (기본 옵션)
- --hard : index 취소, 워킹트리 취소

## commit 취소

#### push 하지 않은 경우(commit 만 한 상태)

최종 커밋을 취소. _워킹트리는 보존됨_

```bash
git reset HEAD^
```

마지막 2 개의 커밋을 취소. _워킹트리는 보존됨_

```bash
git reset HEAD~2
```

마지막 2 개의 커밋을 취소. _index 및 워킹트리 모두 원복됨_

```bash
git reset --hard HEAD~2
```

머지한 것을 이미 커밋했을 때, 그 커밋을 취소. (잘못된 머지를 이미 커밋한 경우)

```bash
git reset --hard ORIG_HEAD
```

#### push 한 경우

HEAD 에서 변경한 내역을 취소하는 새로운 커밋 발행(undo commit).

```bash
git revert HEAD
```

## diff

#### 최근 commit 과 현재 파일 비교

```bash
git diff HEAD
```

1. staged 상태만

   전체 파일 비교

   ```bash
   git diff --cached
   ```

   특정 파일 비교

   ```bash
   git diff --cached 파일이름
   ```

2. unstaged 상태만

   ```bash
   git diff
   ```

#### 변경된 라인이 아니라 word 를 보고 싶을 때

```bash
git diff --color-words
git diff --word-diff
```

#### commit 간 비교

```bash
git diff hash1 hash2
```

#### 브랜치간 비교

```bash
git diff 브랜치 1 브랜치 2
```

#### 최근 commit 과 이전 commit 비교

```bash
git diff HEAD HEAD^
```

## log

```bash
git log
```

git log 명령어는 시간 순으로 commit 이력을 보여준다.

#### commit 의 상세한 변경 내용을 보고 싶을 때(n 개의 커밋에 대해 (생략가능))

```bash
git log -p -n
```

## patch

#### 특정 Commit ID 를 지정하여 Patch 파일 만들기

```bash
git format-patch {Commit ID}
```

#### 최근 Commit 부터 n 개의 Commit 을 Patch 파일 생성

```bash
git format-patch -n
```

#### diff 를 이용한 patch 파일 생성

```bash
git diff > {Patch 파일명}
```

#### Patch 적용

```bash
git am {Patch 파일명}
```
