---
description: >-
  在gitlab项目中，使用了太久的项目会有很多的提交，项目非常地大，如何清空项目以前的提交呢？又能保证现在文件不受影响，这里提供一个思路及解决方案，就是使用一个新创建的分支来处理这一情况。
---

# 清理Gitlab项目节省空间

克隆一个项目

```text
git clone ssh://git@gitlab.demo.com:10022/lake/aa.git
cd aa
```

创建临时分支

```text
git checkout --orphan tmp
```

添加所需要的文件

```text
git add -A
```

添加commit信息

```text
git commit -m "clean project"
```

删除master分支

```text
git branch -D master
```

更名分支

```text
git branch -m master
```

提交分支

```text
git push -f origin master
```

## 异常

第一次使用会出现以下错误

```text
To ssh://git@gitlab. demo.com:10022/lake/aa.git
 ! [rejected]        master -> master (non-fast-forward)
error: 无法推送一些引用到 'ssh://git@gitlab. demo.com:10022/lake/aa.git'
提示：更新被拒绝，因为您当前分支的最新提交落后于其对应的远程分支。
提示：再次推送前，先与远程变更合并（如 'git pull ...'）。详见
提示：'git push --help' 中的 'Note about fast-forwards' 小节。
[lake@localhost aa]$ git push origin master --force
对象计数中: 5, 完成.
Delta compression using up to 8 threads.
压缩对象中: 100% (2/2), 完成.
写入对象中: 100% (5/5), 271 bytes | 0 bytes/s, 完成.
Total 5 (delta 0), reused 4 (delta 0)
remote: GitLab: You are not allowed to force push code to a protected branch on this project.
To ssh://git@gitlab. demo.com:10022/lake/aa.git
 ! [remote rejected] master -> master (pre-receive hook declined)
error: 无法推送一些引用到 'ssh://git@gitlab. demo.com:10022/lake/aa.git'
```

可在`gitlab`项目的`Settings` -&gt; `Repository` -&gt; `Protected branch` -&gt; `unprotect`

