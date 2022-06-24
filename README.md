# 一、前提准备

1、分支命名规范

dev：开发分支
master：生产发布分支

dev、master 上只进行 merge，不进行变更

feat-x.y.z：功能特性开发分支
fix-x.y.z：热修复分支
release-x.y.z：预发布分支

2、分支合并

git rebase -i：指定基底重做
git merge
 * --ff：Fast Forward
 * --no-ff：No Fast Forward

# 二、基础操作

## 1、初始化

1、master 初始化

 * READMD.md
 * .gitignore
 * Maven 框架

2、从 master 拉取 dev，dev 初始化

 * 项目框架
 * 修改 pom.xml 版本号，commit "Release x.y.z"

## 2、开发

1、从 dev 拉取 feat 1..n

 * 开始：进行 新功能开发，开发包括测试用例
 
 * feat 上只做变更，不修改 pom.xml 版本号

 * 结束：feat 上本地测试用例测试完成，可以开始准备合并 feat 到 dev

   * feat 1..n rebase -i dev

2、dev merge --ff feat 1..n

 * dev 每次 merge feat 后都修改 pom.xml 版本号，commit "Release x.y.z"

## 3、预发布（ 联调测试 ）

1、从 dev 拉取 release，开始进行 release，**此时 dev 被加锁**

 * 开始：release 上只进行 Bug 修复、文档生成 和 其他面向发布的任务
 
 * release 可以被手动 CICD 到 DEV/UAT 环境，进行联调测试
 
 * release 每次变更提交同时修改 pom.xml 版本号

 * 结束：release 上联调测试完成，可以开始合并 release 到 dev、master

2、dev merge --ff release，**此时 dev 被解锁**

## 4、发布

1、master merge --no-ff release

 * 开始：master 每次 merge release 都需要根据 pom.xml 版本号打上 tag
   * tag vx.y.z HEAD
 * 结束：master 可以被手动 CICD 到 PRD 环境，进行生产发布

## 5、热修复

1、从 master 拉取 fix 1..n

 * 开始：进行 生产 Bug 热修复，每次变更提交同时修改 pom.xml 版本号
 * 结束：fix 上修复完成，可以开始准备合并 fix 到 master
   * fix 1..n rebase -i master

2、master merge --ff fix 1..n

 * 开始：master 每次 merge fix 都需要根据 pom.xml 版本号打上 tag
   * tag vx.y.z HEAD
 * 结束：master 可以被手动 CICD 到 PRD 环境，进行生产发布

3、dev | release merge --no-ff fix 1..n

 * dev | release 每次 merge fix 都会有 pom.xml 版本号的变更冲突，需要手动进行修复

# 三、高级操作

## 1、git stash 同步

stash 分支不支持多人协作，仅供个人使用

使用场景：下班时将开发到一半的代码 " stash 上传 " ，回到家中进行 " stash 下载 "

### 1.1、stash 上传

1、从 ${branch} 拉取 ${branch}-stash 分支

2、git add . 添加 stash 内容并提交，git commit msg 类型为 " Stash "，最后进行 push

> 注：git push 前检查若存在 origin/${branch}-stash 分支，说明已存在的 stash 未下载，需要进行下载合并

3、切换回 ${branch} 分支，并删除本地 ${branch}-stash 分支

### 1.2、stash 下载

1、切换到相同分支，git fetch origin ${branch}-stash 分支

> 注：若不存在 origin/${branch}-stash 分支，说明当前无 stash 需要同步

2、git cherry-pick origin/${branch}-stash，下载 stash 到当前分支

3、git reset HEAD~，还原 stash，确认无误后删除 origin/${branch}-stash 分支

4、基于 stash，继续进行开发和 commit，若有新的 stash 需要同步，则重复上传的操作

## 2、多版本并行

1、并行多版本产生

 * 架构发生更改，版本号变化：x.y.z -> { x + 1 }.0.0

 * 原 x.y.z 版本仍需要继续使用

2、老版本 dev、master 更名后继续迭代

 * dev 重命名为 dev-{x}.x
 
    `git checkout -b dev-{x}.x dev`
 
 * master 重命名为 branch-{x}.x
 
    `git checkout -b branch-{x}.x master`

> 注：feat-x.y.z、release-x.y.z、fix-x.y.z 命名不变

3、新版本沿用 dev、master

> 注：注意 pom.xml 文件同步修改版本号