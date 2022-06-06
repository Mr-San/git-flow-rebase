# 前提准备

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

# 初始化

1、master 初始化

 * READMD.md
 * .gitignore
 * Maven 框架

2、从 master 拉取 dev，dev 初始化

 * 项目框架
 * 修改 pom.xml 版本号，commit "Release x.y.z"

# 开发

1、从 dev 拉取 feat 1..n

 * 开始：进行 新功能开发，开发包括测试用例
 
 * feat 上只做变更，不修改 pom.xml 版本号

 * 结束：feat 上本地测试用例测试完成，可以开始准备合并 feat 到 dev

   * feat 1..n rebase -i dev

2、dev merge --ff feat 1..n

 * dev 每次 merge feat 后都修改 pom.xml 版本号，commit "Release x.y.z"

# 预发布（ 联调测试 ）

1、从 dev 拉取 release，开始进行 release，**此时 dev 被加锁**

 * 开始：release 上只进行 Bug 修复、文档生成 和 其他面向发布的任务
 
 * release 可以被手动 CICD 到 DEV/UAT 环境，进行联调测试
 
 * release 每次变更提交同时修改 pom.xml 版本号

 * 结束：release 上联调测试完成，可以开始合并 release 到 dev、master

2、dev merge --ff release，**此时 dev 被解锁**

# 发布

1、master merge --no-ff release

 * 开始：master 每次 merge release 都需要根据 pom.xml 版本号打上 tag
   * tag vx.y.z HEAD
 * 结束：master 可以被手动 CICD 到 PRD 环境，进行生产发布

# 热修复

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
