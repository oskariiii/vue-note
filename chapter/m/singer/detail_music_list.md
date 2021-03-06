# 歌手详情页-music-list组件

歌曲列表组件开发，还要啰嗦一句，第一次开发你肯定不知道哪些是需要组件的，所以这个组件的抽象，要么是做之前，看了一个整体的设计图，然后分离出来的，观察他们的表现形式，是否类似，结构是否类似，如果是的话就可以做成组件了。

## 骨架页
![](/assets/musicapp/歌手详情歌曲列表页面骨架效果.png)

先分析下页面结构和实现思路：

1. 一个返回按钮
2. 一个标题
3. 一个背景图
4. 一个歌曲列表

实现思路：

1. 返回按钮 ：小图标加绝对定位
2. 标题 ：绝对定位 + 宽度百分之80 + left 10% + 文字居中，正好让标题块居中
3. 背景图 ： background-image:url() + 一个div模拟的半透明滤镜 + 绝对定位覆盖掉背景图片

上面html和css要注意的是，因为dom结构的顺序，所以要注意z-index 的值，因为后来居上，所以要设置z-index的值不然就看不到返回按钮和标题了

**src/components/music-list/music-list.vue**
```html
<template>
  <div class="music-list">
    <div class="back">
      <i class="icon-back"></i>
    </div>
    <h1 class="title" v-html="title"></h1>
    <div class="bg-image" :style="bgStyle">
      <div class="filter"></div>
    </div>
  </div>
</template>
```
```javascript
<script type="text/ecmascript-6">
  export default {
    data () {
      return {}
    },
    props: {
      // 背景图
      bgImage: {
        type: String,
        default: ''
      },
      // 歌曲列表
      songs: {
        type: Array,
        default: []
      },
      // 歌手名称
      title: {
        type: String,
        default: ''
      }
    },
    computed: {
      bgStyle () {
        return `background-image:url(${this.bgImage})`
      }
    }
  }
</script>
```
```css
<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"
  @import "~common/stylus/mixin"

  .music-list {
    position fixed
    z-index 100
    top 0
    left 0
    bottom 0
    right 0
    background $color-background
    .back {
      position absolute
      top: 0
      left: 6px
      z-index: 50
      .icon-back {
        display block
        padding 10px
        font-size $font-size-large-x
        color: $color-theme
      }
    }
    .title {
      position absolute
      top 0
      left 10% // 宽度百分之80，左边10，右边10也就居中了
      width 80%
      z-index 40
      no-wrap()
      text-align center
      line-height 40px
      font-size $font-size-large
      color $color-text
    }
    .bg-image {
      position relative
      width 100%
      height 0
      padding-top 70%
      transform-origin top
      background-size cover
    // 滤镜效果，图片用背景做
    // 滤镜用div定位做
      .filter {
        position absolute
        top 0
        left 0
        width 100%
        height 100%
        background rgba(7, 17, 27, 0.4)
      }
    }
  }
</style>
```

## 列表的页的引用
引入 [歌手详情页-song-list组件的 - 编写列表组件 小节中的第一个版本列表](/chapter/m/singer/detail_song_list.md)
因为还用到了滑动，所以还要引入基础组件的scroll组件

```html
.....
<scroll :data="songs" class="list" ref="list">
      <div class="song-list-wrapper">
        <song-list :songs="songs"></song-list>
      </div>
    </scroll>
```
```javascript
script type="text/ecmascript-6">
  import SongList from 'base/song-list/song-list'
  import Scroll from 'base/scroll/Scroll'

  export default {
    components: {
      Scroll, SongList
    },
    ......
    mounted () {
      // 因为这是一个scroll组件，所以要拿到el（dom元素）
      // 为什么要动态计算高度呢？是因为背景图的高度是自适应的
      this.$refs.list.$el.style.top = `${this.$refs.bgImage.clientHeight}px`
      console.log(this.$refs.list)
    }
```
```css
....
 .list {
      position: fixed
      top: 0
      bottom: 0
      width: 100%
      background: $color-background
      overflow hidden
      .song-list-wrapper {
        padding: 20px 30px
      }
      .loading-container {
        position: absolute
        width: 100%
        top: 50%
        transform: translateY(-50%)
      }
    }
```


死板的列表样子是做出来的。
接下来我们要实现一个动效（类似原生的app交互效果）：

1. 希望在列表往下滑动（手指会往上滑动）的时候，上面的背景图局域能随着滑动的上升而挤走
2. 列表往上滑动（手指会往下滑动）的时候，背景图随着滑动的下降而逐渐显示出来

## 交互效果
先来手指往上滑动，歌曲列表会滚动到距离页面顶部的一定距离处，然后固定
![](/assets/musicapp/歌手详情歌曲列表交互效果图顶部问题.png)

实现思路：

1. 去掉了list的overflow hidden，能让我们滚动出容器，但是容器背景不会跟着滚动
2. 所以我们需要另外一个层与容器背景颜色一致的，滚动的时候把这个层跟着滚动，看起来的效果就是这个列表往上移动了。
  
    为什么能让这个层跟着滚动还能做列表的背景呢？因为css样式，列表是position: fixed布局，top是js根据上面的背景图片部分计算出来的。所以是悬浮的。而层是普通的div元素。使用translate3d让div元素偏移。
```

```html
<template>
  <div class="music-list">
    <div class="back">
      <i class="icon-back"></i>
    </div>
    <h1 class="title" v-html="title"></h1>
    <div class="bg-image" :style="bgStyle" ref="bgImage">
      <div class="filter"></div>
    </div>
    <div class="bg-layer" ref="bgLayer"></div>
    <scroll :data="songs" class="list" ref="list"
            :listen-scroll="listenScroll"
            :probe-type="probeType"
            @scroll="listScroll"
    >
      <div class="song-list-wrapper">
        <song-list :songs="songs"></song-list>
      </div>
    </scroll>
  </div>
</template>
```
```javascript
    created () {
      this.listenScroll = true  // 监听滚动事件
      this.probeType = 3  // 滚动事件派发方式
    },
    mounted () {
      // 因为这是一个scroll组件，所以要拿到el（dom元素）
      // 为什么要动态计算高度呢？是因为背景图的高度是自适应的
      let clientHeight = this.$refs.bgImage.clientHeight
      this.$refs.list.$el.style.top = `${clientHeight}px`

      // 记录两个值，背景图部分的高度
      this.bgImageHeight = clientHeight
      // 交互列表能偏移的最小高度是
      this.minTranslateY = -(clientHeight - RESERVED_HEIGHT)
    },
    data () {
      return {
        scrollY: 0
      }
    },
    methods: {
      listScroll (pos) {
        this.scrollY = pos.y
      }
    },
    watch: {
      scrollY (newY) {
        // 当滚动到顶部距离 背景图片的高度剪掉我们要留出来的高度的时候就让背景层不滚动了
        // 或则是根本就是手指往下滑动的时候，滚动到０的时候也就是原来没有滚动的样子，也不再滚动背景层了
        //        let top = this.$refs.bgImage.clientHeight - 40
        //        if (newY < -top || newY > 0) {
        //          // 在不滚动的同时，我们还要把列表容器的overflow hidden加上，就不会让歌曲列表滚动到容器外部了
        //          this.$refs.list.style['overflow'] = 'hidden'
        //          return
        //        }
        /** --------------
         * 上面说的思路直接作废:因为我们没有移动列表，只是让一个div层跟着滚动的,
         * 所以：如果改变成overflow＝hidden，那么这个列表就又不能滚动到上面来了
         * ---------- */

        let translateY = Math.max(this.minTranslateY, newY)
        if (translateY === this.minTranslateY || translateY === 0) {
          return
        }
        // 让背景层跟着网上移动
        this.$refs.bgLayer.style['transform'] = `translate3d(0,${translateY}px,0)`
        console.log(newY)
      }
```

## [顶部列表滚动出容器问题解决](/chapter/m/singer/detail_music_list2.md)

