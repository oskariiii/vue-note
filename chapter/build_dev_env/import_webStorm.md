# 使用webStorm开发
## 开发工具安装
本次使用的版本是：JetBrains WebStorm 2016.3
安装自行百度。熟悉idea的就更容易了。因为和idea的很多很多配置都是一致的。只是该工具针对前端开发。

## 导入项目
直接打开项目根目录，导入ok。

随意打开一个.vue文件.一般会报错。在报错的地方使用Alt+回车 选择更换成es6语法。
```javascript
<script>
export default
```

## 项目运行
```
Run -> Edit Configurations -> + -> npm 
```
选择npm后，会出现一个配置界面，如下图选择![](/assets/image/B7B5568901C17601B6792A5C178434E5.jpg)

保存配置后，就会在Run菜单中出现 你的配置。选择并运行Run 'npm_run'

## EsLint 设置
在`.eslintrc.js` 中继承了 该规则 https://github.com/feross/standard/blob/master/docs/RULES-zhcn.md

```javascript
  // https://github.com/feross/standard/blob/master/RULES.md#javascript-standard-style
  extends: 'standard',
```

有些版本的WebStorm没有打开EsLint的选项。需要经过以下方式打开：
![](/assets/image/eslint-config.png)

**特别注意的是：**有些版本或则有时候抽风配置后能检测到，但是格式化的时候不能按照eslint的规则进行格式化，那么可以看看以前可以格式化项目的 `\.idea\codeStyles\Project.xml` 文件中的 `JSCodeStyleSettings` 和 `codeStyleSettings` 中是否有相关的配置。 暂时我还没有搞清楚为什么有时候有效果有时候没有

这是一个基础环境搭建后，被正确读取出来的配置；包含了下面的设置成2个空格的配置。其他的都是自动读取的。
如果你的webStorm没有自动读取，并且格式化不会按照eslint规范来格式化，可以尝试覆盖这个文件（如果没有则新建）
```xml
<component name="ProjectCodeStyleConfiguration">
  <code_scheme name="Project" version="173">
    <JSCodeStyleSettings>
      <option name="USE_SEMICOLON_AFTER_STATEMENT" value="false" />
      <option name="FORCE_SEMICOLON_STYLE" value="true" />
      <option name="SPACE_BEFORE_GENERATOR_MULT" value="true" />
      <option name="USE_DOUBLE_QUOTES" value="false" />
      <option name="FORCE_QUOTE_STYlE" value="true" />
      <option name="SPACES_WITHIN_IMPORTS" value="true" />
    </JSCodeStyleSettings>
    <MarkdownNavigatorCodeStyleSettings>
      <option name="RIGHT_MARGIN" value="72" />
    </MarkdownNavigatorCodeStyleSettings>
    <codeStyleSettings language="JavaScript">
      <option name="KEEP_BLANK_LINES_IN_CODE" value="1" />
      <option name="SPACE_BEFORE_METHOD_PARENTHESES" value="true" />
      <option name="TERNARY_OPERATION_SIGNS_ON_NEXT_LINE" value="true" />
      <option name="KEEP_SIMPLE_BLOCKS_IN_ONE_LINE" value="true" />
      <option name="KEEP_SIMPLE_METHODS_IN_ONE_LINE" value="true" />
      <indentOptions>
        <option name="INDENT_SIZE" value="2" />
        <option name="CONTINUATION_INDENT_SIZE" value="2" />
        <option name="TAB_SIZE" value="2" />
      </indentOptions>
    </codeStyleSettings>
  </code_scheme>
</component>
```

## 分号
打开以后的最明显的效果就是：以上的默认规则是 不需要分号的。 使用代码格式化后，会自动把分号都去掉！！听说在官网规则中有小扳手图标的都能自动按规则修复。

## 缩进两个空格
![](/assets/image/缩进2格设置.png)

## 热加载
热加载配置在`build/dev-server.js`文件中定义的。相关配置是`devMiddleware`,`hotMiddleware` 相关的。

这个热加载配置的作用就是：
大部分配置修改之后，浏览器会自动刷新页面/或局部刷新，展示最新的内容的功能。

在webStorm中有一个选项会导致该配置不生效，不能自动刷新。
```
File -> settings -> System Settings -> Synchronization -> Use "safe write"(save changes to a temporary file first)

把该选择去掉。
```

然后你尝试在App.vue `template`中增加一些文字内容。会在控制台出现以下提示。表示热加载成功了。
```bash
> Listening at http://localhost:8080

c WAIT  Compiling...

c DONE  Compiled successfully in 556ms
```

## 小技巧 - 预定义一个Vue文件模版
.vue 文件中一般都是以下三个模版内容。所以我们把它设置为文件模版。就可以省略每次都敲这些固定的模版了。
```javascript
<template>
    <p> template </p>
</template>

<script>
  export default {
      data () {
         return {};
      }
  };
</script>

<style lang="less" rel="stylesheet/less" scoped>

</style>
```
在以下界面中新增一个模版。
```
File -> Settings -> Editor -> File and Code Templates
```
设置如下图：
![](/assets/image/414344CED59ADD904753A6833A6920AC.jpg)

在新建文件的时候选择我们已经配置好的 Vue File项。

