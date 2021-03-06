# vue组件库开发

组件库开发的目的：现在公司小项目可能会比较多，虽然用的element ui 但是还有一部分会根据自身需要进行封装增强或则针对业务化公用的封装，这样一来就会存在以下几个问题：

1. 每个项目中都有部分通用公用的组件，要么都copy一次
2. 维护困难，如果有bug或则新增功能，所有项目都需要重新copy一次

如果不分那么细致的话，可以连同工具类一起发布到npm中，只要所有项目依赖该组件库，就能解决上面的维护问题。

依照够用原则，放弃了element ui的构建方式，原因是学习成本比较高，后来发现 iview 组件库的方式和vue-cli生成的配置差不多，就依此为参考进行开发环境的搭建；

## 环境搭建思路
使用vue-cli为基础，安装好后，再修改配置文件；
百度看了好多讲解打包组件的方式，但是都是很简单配置和vue-cli脱节的的开发方式，可能学习成本来说会让人摸不着头脑。 参考了一些开源框架，最后终于大概弄明白了一些概念：

- 开发环境和vue-cli默认生成的配置类似
- 不同的是 组件的打包，要打包成js文件 和 css文件。
- 发布到npm的内容是.npmignore 和 package.json决定的

以上几点明白了。就可以尝试搭建了;

**该分支是本教程一步一步实践之后最终成品：**（master会逐步的把一些不需要的代码给去掉，最后形成真正的种子环境项目）
https://github.com/zq99299/vue-components-lib-dev-env/tree/tutorials



