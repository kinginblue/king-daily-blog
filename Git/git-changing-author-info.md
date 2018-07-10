# Git 更改作者信息

## 一|问题引出

有时候，我们的 Git Repository 提交了很长的历史了，突然有一天发现，提交信息里不是我们想要的 `username` 和 `email`。

比如在工作电脑上， `global` 的 `git config` 信息是公司的，这些信息被当作 `author` 提交到了自己的私人仓库里。又比如有小伙伴刷 `github contribution graph` 的，猛然回头一看，`contribution graph` 没显示出来。等等

## 二|如果一切还没开始

如果是新 Repository 则好办，在 Repository 根目录下的 `.git` 目录下，找到 `config` 文件，并在其中加上自己想要的 `author` 信息，那么这个 Repository 的 `author` 信息就会覆盖 `global` 的 `author` 信息

```config
[user]
    name = your-name
    email = your-email@email.com
```

或者直接在 Repository 下命令行：

```bash
git config user.email "your-email@email.com"
git config user.name "your-name"
```

## 三|官方解决方案

原文 [Changing author info](https://help.github.com/articles/changing-author-info/)

但更一般的是，我们的 Repository 提交了很多次，并且已经 `push` 到了远程仓库。

官方给出了一个脚本，按照以下步骤操作一遍，就可以了。

    注意：执行这段脚本会重写 repo 所有协作者的历史。完成以下操作后，任何 fork 或 clone 的人必须获取重写后的历史并把所有本地修改 rebase 入重写后的历史中。

    在执行这段脚本前，你需要准备的信息：
    ·欲修改的旧的邮箱地址
    ·正确的用户名和邮箱地址

1.打开终端（Mac 或 Linux 用户）或命令行（Windows 用户）。

2.创建一个你的 repo 的全新裸 clone

```bash
git clone --bare https://github.com/user/repo.git
cd repo.git
```

3.复制粘贴脚本，并根据你的信息修改以下变量：

    OLD_EMAIL
    CORRECT_NAME
    CORRECT_EMAIL

```sh
#!/bin/sh

git filter-branch --env-filter '

OLD_EMAIL="your-old-email@example.com"
CORRECT_NAME="Your Correct Name"
CORRECT_EMAIL="your-correct-email@example.com"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

4.按 `Enter` 执行脚本。

5.查看新 Git 历史有没有错误。

6.把正确历史 push 到 Github

```bash
git push --force --tags origin 'refs/heads/*'
```

7.清除临时 clone。

```bash
cd ..
rm -rf repo.git
```
