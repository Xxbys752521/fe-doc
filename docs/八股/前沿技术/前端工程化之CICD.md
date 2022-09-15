   

[![](https://p3-passport.byteimg.com/img/user-avatar/abce2cc1c1b008537a9115eb7de44604~100x100.awebp)](https://juejin.cn/user/712139267910237)

2020年09月09日 11:30 ·  阅读 8091

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a010befd5dc4f5cbfb8080f65cf967a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

> 前沿：朋友们，你还在手动“丢包”吗？机械化搬运工当得不是滋味吧？想不想学习自动化流水线构建～如果想，这篇适合你，结合CICD来自动化构建前端项目，本文树酱🌲主要介绍如何基于jenkins和travis的基础上让 CI/CD 跑起来，解放你的双手👋，从此告别996

## 1.远古时代

> 我们知道，对于一般的SPA应用，本质是静态资源（后端渲染SSR忽略），执行build命令，把项目打包build一下完，压缩打包之后的文件，ssh连接服务器并把压缩好的文件“丢”到服务器，解压上传的文件，最后配置下Nginx即可访问到该项目的资源，石器时代我们是这样走流程的，流程如下

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13ec4170218d4855b5b87fa5bfa5df54~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

我们大概需要完成以下操作

-   本地执行 `npm run build` 构建项目，压缩编译好的资源文件
-   将压缩包丢到远程服务器
-   ssh到远程服务器，解压压缩包
-   配置nginx

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d7f6da255a1b4d5d89b359821520b114~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

## 2\. 铁器时代

> 到后来前端有了自己的工具链，为了让发布前对代码健壮性和功能完整性有个验证，在发布流程中加入了单元测试和代码扫描，验证完之后再通过服务器手动拉取最新代码（git）再build编译项目，最后配置下Nginx即可访问到该项目的资源，铁器时代我们是这样走流程的，流程如下

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40bd96fcb31c480e8757f32616533619~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

完成一个完整的前端项目发布闭环，我们大概需要完成以下操作

-   代码扫描 `npm run lint` 检查代码是否规范（`eslint`）
-   本地跑单元测试 `npm run unit` 检查单元测试结果
-   用`git`将测试完的代码提交到远程仓库如`gitlab`
-   登录远程测试服务器，拉取代码，执行 `npm run build` 构建项目
-   如果是后端渲染项目（SSR）如果是基于pm2做进程管理的还需要重启 `pm2 restart`

每次发布都需要手动“丢包”，不断重复机械化的工作，可想而知效率会有多慢，而且更难保证每次每个步骤都不会疏忽，可能忘记做单元测试就进行了代码提交，造成程序出错等

> 思考：👨🎓啊乐同学：那我可以将这些细节把命令集成到一个shell脚本呀，这样也不是自动化？

## 3.CICD时代

> CICD是什么？ 顾名思义就是持续集成(Continuous Integration)和持续交付(Continuous Delivery),简单理解就是把我们之前需要手动去执行的部署构建环节自动化，一步到位，解放双手

> 👨🎓 啊宽同学：还是有点搞不懂持续集成和持续交付的区别是什么

-   持续集成：当代码仓库代码发生变更，就会自动对代码进行测试和构建，反馈运行结果。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e27324e561114ef6adb461f664dbfb64~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

-   持续交付：持续交付是在持续集成的基础上，可以将集成后的代码依次部署到测试环境、予发布环境、生产环境等中

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fad33566623e477b8db232a8b717d6b4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

🌲拓展阅读：

-   [全面回答什么是持续集成和持续交付](https://link.juejin.cn/?target=http%3A%2F%2Fwww.uml.org.cn%2Fjchgj%2F202002194.asp "http://www.uml.org.cn/jchgj/202002194.asp")

那么我们有什么工具可以帮助我们来完成这一系列操作呢？平时中我用的比较多的两种方式：`Jenkins CI/CD` 和 `Travis CI`

### 3.1 Travis CI

> Travis CI是持续集成服务的实现方式之一，不过它跟GitHub有点“捆绑销售”的样子，傻傻分不开，因为Travis只支持github、gitlab等代码托管平台。那么Travis是如何做持续集成的呢，只要代码仓库有新的代码变更，就会自动抓取然后完成测试和构建，下面🌲酱通过搭建一个github项目实操来介绍“Travis”的正确使用姿势，附上官网链接[🔗Travis-ci](https://link.juejin.cn/?target=travis-ci.org "travis-ci.org")

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a09c8bddb7cf4f4d8d3199c2a9035dcd~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 3.1.1 准备工作

1.需要在travis-ci.org注册好你的专属travis-ci账号，然后绑定你的github，登陆后选择你要集成的项目

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37617dd55c9c4496a4bc86120110aa63~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

2.完成上述操作之后，在你想要做持续集成的项目根目录中创建一个文件`.travis.yml`，这个文件的意义在于用来预先定义好Travis的行为。 当代码仓库有新的Commit时，Travis会去项目根目录寻找该文件并执行里面的命令，我们看看树酱定义好的`.travis.yml`

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c1a7281cd7e439687df977ca1c4c486~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

以上的定义主要由以下这些主要配置组成 👇

-   `language`：字段指定了默认运行环境
-   `node_js`: 用来指定 Node 版本。
-   `install`：用来指定安装脚本或依赖
-   `script`：运行脚本

`install阶段`和`script阶段`，这里要区分一个细节：

-   如果是install阶段中的其中一个任务失败，则整个任务中止，整个构建阶段的状态也是失败。
-   如果是script阶段中的其中一个任务失败，则任务进行，构建阶段的状态跟install一样也是失败

3.当代码仓库中代码发生变更，Travis就会自动触发，并执行你`.travis.yml`定义好的命令，完成测试和构建 ![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/216cf03d031d4a728155000e37f1a60d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 3.1.2 实操使用

> 👩🎓啊乐同学：树酱如果是CI过程中出错是怎么样的情况？

项目在构建与测试多多少少会出现失败的情况，下面是一个实际的单元测试出错例子，一旦出错则中断CI行为（因为树酱将单元测试命令配置在install阶段）

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7519191d61a47a09ede85032c3b35e0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df892a86eb3746d4ae01e2b7133cfd2f~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

以上是一个简单的travis实现自动化集成的小demo，Travis能做的事情还很多，比如构建你的Page Github等等

🌲拓展阅读：

-   [使用 travis + gitbook + github pages 优雅地发布自己的书](https://link.juejin.cn/?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000018002602%3Futm_source%3Dtag-newest "https://segmentfault.com/a/1190000018002602?utm_source=tag-newest")
-   [持续集成服务 Travis CI 教程](https://link.juejin.cn/?target=http%3A%2F%2Fwww.ruanyifeng.com%2Fblog%2F2017%2F12%2Ftravis_ci_tutorial.html "http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html")

### 3.2 Jenkins CI/CD

> 上一节我们介绍了travis，也知道travis依赖github的代码仓库管理，那如果万一公司内部使用的是svn而不是git呢？虽然git的现在是主流，jenkins拓展性比较强，git、svn都支持。同时jenkins作为一个可扩展的自动化服务器，可以用作简单的 CI 服务器，具有自动化构建、测试和部署等功能，简而言之，jenkins可以方便我们日常的前端项目版本更新迭代（开发、测试、生产环境等），也可以通过它自动化完成一系列的操作包括：编译打包元测试、代码扫描等

下面通过介绍两种构建配置来构建：默认的配置和流水线配置

#### 3.2.1 模式一：默认的配置修改

-   Source Code Management，首先是代码仓库的配置

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/140c65b1cd0447caaa58a0c0e26beea0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

-   Build Triggers

> 选择build的触发模式，默认是手动触发，支持代码触发构建和定时构建

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/91e789c1171f46f4ae4e1977e8a181c3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

-   build 命令

> 选择执行的脚本命令

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/097bb4de67c24aec99572bdc66f0bbe0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

-   Post-build Actions

> 主要是用于多节点时需要远程，用于集群部署 可添加多台机器远程访问，将build后打包的资源上传到多个节点更新资源

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1128dbe0cde74e31bc438c25e4ca44bf~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 3.2.2 模式二：jenkins流水线配置

> 这里主要介绍jenkins流水线配置的使用，流水线的代码定义了整个的构建过程, 他通常包括构建, 测试和交付应用程序的阶段，下面是路径和仓库的配置

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56e11513504c4b0f8f583a322461e294~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

图片相关配置如下：

-   SCM:选择git或者svn作为代码触发器
-   脚本路径：在项目根目录创建jenkinsfile来编写流水线

下面介绍一个简单版的jenkinsfile的流水线任务写法，完成整个前端工程化部署涉及的编译打包、静态扫描、单元测试等环节

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9cd00e843b42440bb55794dc12a5c6b3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

完成后，即可构建项目，分阶段完成，首先是下拉源码、代码构建编译、代码扫描等等，所有环节成功才算自动化部署成功，如下所示

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac105c1624a64a8db645c024e5db43c0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

🌲酱往期文章：

-   [树酱的前端知识体系构建（上）](https://juejin.cn/post/6855468132186882055 "https://juejin.cn/post/6855468132186882055")
-   [树酱的前端知识体系构建（下）](https://juejin.cn/post/6860018724221976584 "https://juejin.cn/post/6860018724221976584")
-   [聊聊前端开发日常的协作工具](https://juejin.cn/post/6844904176330375181 "https://juejin.cn/post/6844904176330375181")
-   [babel配置傻傻看不懂](https://juejin.cn/post/6863705400773083149 "https://juejin.cn/post/6863705400773083149")
-   [如何更好管理 Api 接口](https://juejin.cn/post/6844904154574356493 "https://juejin.cn/post/6844904154574356493")
-   [面试官问你关于node的那些事](https://juejin.cn/post/6844904177466867726 "https://juejin.cn/post/6844904177466867726")
-   [前端工程化那些事](https://juejin.cn/post/6844904132512317453 "https://juejin.cn/post/6844904132512317453")
-   [你学BFF和Serverless了吗](https://juejin.cn/post/6844904185427673095 "https://juejin.cn/post/6844904185427673095")
-   [前端运维部署那些事](https://juejin.cn/post/6844904118020997128 "https://juejin.cn/post/6844904118020997128")

#### 请你喝杯🍵 记得三连哦～

1.阅读完记得给🌲 酱点个赞哦，有👍 有动力

2.关注公众号前端那些趣事，陪你聊聊前端的趣事

3.文章收录在Github [frontendThings](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FlittleTreeme%2FfrontendThings "https://github.com/littleTreeme/frontendThings") 感谢Star✨

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/00ba359ecd0075e59ffbc3d810af551d.svg) 142

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/3d482c7a948bac826e155953b2a28a9e.svg) 收藏