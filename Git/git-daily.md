# Git 日常

## 显示中文路径名/文件名

```bash
git config --global core.quotepath false
```

## git clone 到指定目录

```bash
git clone the-repo.git the-path
```

## git clone 时不带历史信息，方式一

`clone` 下来后，手动清除 `.git` 目录后重新 `git init`。这在 `clone` 下来的仓库还需 push 到其他仓库时很有用：

```bash
git clone the-repo.git
cd the-repo
rm -rf .git
git init

# and push to other repo
git push other-repo.git master
```

## git clone 时不带历史信息，方式二

参考：

- [Copy a git repo without history](https://stackoverflow.com/questions/29368837/copy-a-git-repo-without-history)
- [Clone git repository without history?](https://stackoverflow.com/questions/30001304/clone-git-repository-without-history/30001366)

做不到完全不带历史记录。使用 `--depth` 参数带最近的 n 条历史 commit（n>0）。例如只带最后一条：

```bash
git clone --depth 1 the-repo.git
```

这种模式叫 `shallow clone`。

但是！这样 `clone` 下来的仓库，如果后续需要 `push` 到其他仓库时，就会报错：

```console
! [remote rejected] master -> master (shallow update not allowed)
```

解决方案：[Remote rejected (shallow update not allowed) after changing Git remote URL](https://stackoverflow.com/questions/28983842/remote-rejected-shallow-update-not-allowed-after-changing-git-remote-url)

```bash
git remote add old <path-to-old-remote>

git fetch --unshallow old
```

后续可以：

```bash
# remove the old remote
git remote rm old

# and push to other repo
git push other-repo.git master
```

总的来说，是做不到**方式一**中那样 push 一个不带历史信息的仓库到远端。