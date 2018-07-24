# CentOS yum update 与 yum upgrade 的区别

## 一|辨析

```bash
man yum

update

    If  run  without  any packages, update will update every currently installed package.  If one or more packages or package globs are specified, Yum will only update the listed packages.  While updating packages, yum will ensure that all dependencies are satisfied. (See Specifying package names for more information) If the packages or globs specified match to packages which are not currently installed then update  will  not  install  them.  update  operates  on groups, files, provides and filelists just like the "install" command.

    If  the  main  obsoletes  configure  option  is true (default) or the --obsoletes flag is present yum will include package obsoletes in its calculations - this makes it for distro-version changes, for example: upgrading from somelinux 8.0 to somelinux 9.

    Note that "update" works on installed packages first, and only if there are no matches does it look for available packages. The difference is most noticeable when you do "update foo-1-2" which will act exactly as  "update  foo"  if foo-1-2 is installed. You can use the "update-to" if you'd prefer that nothing happen in the above case.


upgrade

    Is the same as the update command with the --obsoletes flag set. See update for more details.
```

`yum upgrade` 会删除过时的包；而 `yum update` 默认不会删除过时的包，除非加上 `--obsoletes` 参数，作用就和 `yum upgrade` 一样了。

## 二|参考

- man yum
- [In CentOS, what is the difference between yum update and yum upgrade?](https://unix.stackexchange.com/questions/55777/in-centos-what-is-the-difference-between-yum-update-and-yum-upgrade)