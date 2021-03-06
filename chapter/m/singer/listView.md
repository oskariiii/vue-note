# listView歌手列表联/系人列表

## 第一个版本 - 滚动列表
![](/assets/musicapp/listView歌手列表联-系人列表.png)

上面的列表使用了scoll组件，然后渲染了列表
```html
<template>
  <scroll class="list-view" :data="data">
    <ul>
      <li v-for="group in data" class="list-group">
        <h2 class="list-group-title">{{ group.title }}</h2>
        <ul>
          <li v-for="item in group.items" class="list-group-item">
            <!-- 使用了图片的懒加载 --> 
            <img v-lazy="item.avtar" class="avtar">
            <span class="name">{{item.name}}</span>
          </li>
        </ul>
      </li>
    </ul>
  </scroll>
</template>
```
```javascript
<script type="text/ecmascript-6">
  import Scroll from 'base/scroll/Scroll'

  export default {
    props: {
      data: {
        type: Array,
        default: []
      }
    },
    components: {
      Scroll
    },
    data () {
      return {}
    }
  }
</script>
```
```css
<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .list-view {
    width: 100%
    height: 100%
    overflow hidden //这个应该写在父级里面限制这个listview的把。没有明白为什么写在这里也有效果
    background $color-background
    .list-group {
      padding-bottom 30px
      .list-group-title {
        font-size $font-size-small
        line-height 30px
        padding-left 20px
        color $color-text-l
        background $color-highlight-background
      }
      .list-group-item {
        display flex
        align-items center
        padding 20px 0 0 30px
        .avtar {
          width: 50px
          height: 50px
          border-radius: 50%
        }
        .name {
          margin-left 20px
          color $color-text-l
          font-size $font-size-medium
        }
      }
    }
  }
</style>
```

## 右侧快速入口开发
右侧的快速入口，是一个title列表。

![](/assets/musicapp/歌手列表快速列表样式.png)

使用计算属性从列表中拿到所有的title，组成集合
```javascript
    computed: {
      // 快速入口列表集合
      shortcutList () {
        return this.data.map((group) => {
          return group.title.substring(0, 1)  // 热门有两个字，所以要截取掉
        })
      }
    },
```
列表渲染
```html
    <div class="list-shortcut">
      <ul>
        <li class="item" v-for="item in shortcutList">{{ item }}</li>
      </ul>
    </div>
```
```css
.list-shortcut {
      position absolute
      right: 0
      top: 50%
      transform: translateY(-50%)
      width: 20px
      padding 20px 0
      background: $color-background-d
      border-radius: 10px
      text-align: center
      font-family: Helvetica
      .item {
        padding 3px
        color: $color-text-l
        font-size: $font-size-small
        line-height 1
        &.current {
          color: $color-theme
        }
      }
    }
```

## 右侧快速入口 - 联动左侧滚动
点击让列表滚动到指定的title处


我们来分析下这个需求有哪些关键点

1. 快速入口中的每一个title都需要有一个点击事件
2. 让左侧的歌手列表滚动到相应的title处

我们先来解决第一个：点击事件,将h5的触摸事件设置在快速入口的元素上，@touchstart，

```html
<div class="list-shortcut" @touchstart="onShortcutTouchStart">
      <ul>
        <li class="item" v-for="(item,index) in shortcutList"
            :data-index="index">{{ item }}
        </li>
      </ul>
    </div>
```
那么这个元素中的任何一个元素被触摸的时候都会触发该事件。所以这里就有一个问题，怎么才能捕获到只触摸文字的时候（也就是class=item的时候）才触发滚动操作？
```javascript
    methods: {
      // H5 的触摸开始事件
      onShortcutTouchStart (el) {
        // 拿到dom元素在列表中的索引
        let anchorIndex = getData(el.target, 'index')
        if(anchorIndex){
          this.$refs.listview.scrollToElement(this.$refs.listgroup[anchorIndex])
        }
      }
    },
---- getData ----
/**
 * 获取dom 元素上的属性，data开头的属性，
 * @param el
 * @param name
 * @param val 有值就设置，没有值就获取
 * @returns {*}
 */
export function getData (el, name, val) {
  const prefix = 'data-'
  name = prefix + name
  if (val) {
    return el.setAttribute(name, val)
  } else {
    return el.getAttribute(name)
  }
}
```

通过上面的源码可以看到，我们获取触发事件中的dom元素上的'data-index' 属性（在遍历的时候设置在每一个入口文字上的），如果有，就表示需要左侧的内容需要滚动到对应的区域。

然而我们调用了bscroll的一个方法，滚动到指定的元素；
```javascript
this.$refs.listview.scrollToElement(this.$refs.listgroup[anchorIndex])

----scroll.vue 代理了一个bscroll的方法-----
      // 滚动到列表中指定的dom元素位置
      scrollToElement () {
//        this.scroll.scrollToElement(el) 在参数中定义el，和下面的效果一样
        this.scroll && this.scroll.scrollToElement.apply(this.scroll, arguments)
      }
```

这里有组件帮我们做了滚动的操作，那么就怎么找到我们需要滚动到的目标dom元素呢？

这个时候我们想一想，快速列表入口和左侧的列表数据的对应关系是什么？他们的group的标题顺序是一致的，那么我可以通过获取点击的快速列表入口dom元素所在列表中的索引从而找到左侧dom元素所在列表中的索引。 然后我们再给每个group的dom元素一个相同的标识（这里是ref），多个相同的ref组成一个列表，获取列表中指定索引的dom元素，再调用bscroll的scrollToElement就达到了滚动到具体的可视区域


## 右侧快速入口 - 滑动列表- 联动左侧滚动
滑动列表滚动到指定的title处

需求：在快如入口上滑动，变滑动边滚动到对应的title处
难点：

1. 滑动事件
2. 怎么滚动到对应的元素去（也就是怎么计算出需要滚动到哪一个算所索引）
  
基础知识：

1. @touchmove.stop.prevent="onShortcutTouchMove" h5滑动事件，.stop.prevent,vue提供的阻止事件冒泡，因为我们的左侧区域也是有滑动，因为能滚动列表
2. e.touches[0].pageY : 开始触摸处的像素，从0开始，网上滑动5px，则值为-5px，往下滑动5px，值为 5 px

ok上面已经科普完毕：
思路：

1. 引入滑动事件
2. 在滑动起始点（也就是触摸点击事件中）记录这个y点的值
3. 在滑动过程中，再次获取这个y点的值，然后y2-y1 就能知道滚动了多长的距离像素
4. 只要知道一个 字母所在的item元素高度(18)，使用滑动的距离/18，就能知道滑动了几个dom元素
5. 在滑动起始点，还要记录起始点元素的索引，然后使用这个索引 + 第4步骤计算出来滑动的dom元素个数，就能得到当前滚动到的dom元素索引

```
 // H5 的触摸开始事件
      onShortcutTouchStart (el) {
        // 获取到起始点的y值
        let firstTouch = el.touches[0]
        this.touch.y1 = firstTouch.pageY

        // 拿到dom元素在列表中的索引
        let anchorIndex = getData(el.target, 'index')
        if (anchorIndex) {
          this.touch.anchorIndex = anchorIndex
          this.$refs.listview.scrollToElement(this.$refs.listgroup[anchorIndex])
        }
      },
      onShortcutTouchMove (e) {
		// 滑动过程中再次拿到滑动的距离起点的偏移值，往上滑动是负数，往下滑动是正数
        let firstTouch = e.touches[0]
        this.touch.y2 = firstTouch.pageY
        // y 轴上的偏移像素
        let delta = this.touch.y2 - this.touch.y1
        // 得到有几个元素
        delta = delta / ANCHOR_HEIGHT
        // 取整
        delta = delta | 0
		// 然后使用 起点的元素索引，加上滚动的元素个数（因为是上滑的话，得到的值是负数）
		// 假设起点是5，往上滑动-36px个，delta = -2， 5+-2 = 3;所以滚动到的元素索引是正确的
        let anchorIndex = (parseInt(this.touch.anchorIndex) + delta)
        this.$refs.listview.scrollToElement(this.$refs.listgroup[anchorIndex])
        console.log(delta, anchorIndex)
      }
```


## 左侧滚动怎么触发右侧联动高亮对应的title呢？

上面做了右侧滚动或则点击，让左侧定位到对应的块，现在左侧怎么联动右侧呢？

思路：

1. 监听列表的滚动事件
2. 滚动的时候计算当前滚动的区域是属于哪一个group？
    
    那么怎么计算呢？
    1. 在列表数据改变后，记录下每一个group所在列表中的高度
    2. 在滚动时，拿到y轴滚动到的位置，在列表group高度中比较，就能知道当前所属的group是哪一个，得到当前的索引，也就能定位具体的dom元素
    3. 在右侧渲染的时候 :`class="{'current':currentIndex == index}"` 绑定class是当前索引就渲染
    4. 点击右侧的时候没有高亮，点击右侧的时候改变这个索引，就能达到高亮效果了

由于这个组件使用了 基础的 scroll组件，所以要扩展该基础组件的功能，增加监听滚动事件。

bscroll的滚动事件除了下面的代码外，还需要切换probeType=2才会触发滚动的时候派发滚动事件； probeType=1 是滚动结束派发滚动事件，但是我测试的时候没有收到。不知道是啥原因

```javascript
        // 监听bscroll的滚动事件
        if (this.listenScroll) {
          console.log(this.scroll)
          this.scroll.on('scroll', (pos) => {
            this.$emit('scroll', pos)
          })
        }
```

改版完成之后的完整代码
```javascript
<template>
  <scroll class="list-view"
          ref="listview"
          :data="data"
          :listen-scroll="true"
          :probe-type="2"
          @scroll="listScroll">
    <ul>
      <li v-for="group in data" class="list-group" ref="listGroup">
        <h2 class="list-group-title">{{ group.title }}</h2>
        <ul>
          <li v-for="item in group.items" class="list-group-item">
            <img v-lazy="item.avtar" class="avtar">
            <span class="name">{{item.name}}</span>
          </li>
        </ul>
      </li>
    </ul>
    <div class="list-shortcut" @touchstart="onShortcutTouchStart"
         @touchmove.stop.prevent="onShortcutTouchMove">
      <ul>
        <li class="item"
            v-for="(item,index) in shortcutList"
            :class="{'current':currentIndex == index}"
            :data-index="index">{{ item }}
        </li>
      </ul>
    </div>
  </scroll>
</template>

<script type="text/ecmascript-6">
  import Scroll from 'base/scroll/Scroll'
  import { getData } from 'common/js/dom'

  const ANCHOR_HEIGHT = 18  // 每个 list-shortcut 中 item的高度，18=item的字体12 + pading上下3*2

  export default {
    created () {
      // 为什么不在 data里面定义touch？
      // 因为在props中和data中定义的都会被vue监听用于数据的双向绑定
      // 这个并不需要这些监听服务，只需要在当前vue实例中共享数据
      this.touch = {}

      // 配合_calculateHeight方法 存储计算的高度
      this.listHeight = []
    },
    props: {
      data: {
        type: Array,
        default: []
      }
    },
    components: {
      Scroll
    },
    computed: {
      // 快速入口列表集合
      shortcutList () {
        return this.data.map((group) => {
          return group.title.substring(0, 1)  // 热门有两个字，所以要截取掉
        })
      }
    },
    methods: {
      // H5 的触摸开始事件
      onShortcutTouchStart (el) {
        let firstTouch = el.touches[0]
        this.touch.y1 = firstTouch.pageY

        // 拿到dom元素在列表中的索引
        let anchorIndex = getData(el.target, 'index')
        if (anchorIndex) {
          this.touch.anchorIndex = anchorIndex
          this.$refs.listview.scrollToElement(this.$refs.listGroup[anchorIndex])
          // 高亮元素
          this.currentIndex = anchorIndex
          console.log(anchorIndex)
        }
      },
      onShortcutTouchMove (e) {
        let firstTouch = e.touches[0]
        this.touch.y2 = firstTouch.pageY
        // y 轴上的偏移像素
        let delta = this.touch.y2 - this.touch.y1
        // 得到有几个元素
        delta = delta / ANCHOR_HEIGHT
        // 取整
        delta = delta | 0
        let anchorIndex = (parseInt(this.touch.anchorIndex) + delta)
        let length = this.$refs.listGroup.length
        if (anchorIndex >= length) {
          // 由于触摸和scroll事件不同，触摸是按照第一次触摸的点为起点，
          // 这个计算就会超过实际的索引大小
          anchorIndex = length - 1
        }
        this.$refs.listview.scrollToElement(this.$refs.listGroup[anchorIndex], 0)  // 滚动速度，0 是没有滚动动画

        // 高亮元素
        this.currentIndex = anchorIndex
        console.log(delta, anchorIndex)
      },
      listScroll (pos) {
        this.scrollY = pos.y
        console.log(pos)
      },
      // 计算每个group的高度（数据结构是按组进行聚合的）
      _calculateHeight () {
        this.listHeight = []
        const list = this.$refs.listGroup
        let height = 0
        this.listHeight.push(height)
        for (let i = 0; i < list.length; i++) {
          let item = list[i]
          height += item.clientHeight  // 计算每一个group所在的高度区域，从0开始
          this.listHeight.push(height)
        }
      }
    },
    data () {
      return {
        // 监控y轴的位置
        scrollY: -1,
        // 当前索引
        currentIndex: 0
      }
    },
    watch: {
      data () {
        setTimeout(() => {
          this._calculateHeight()
        }, 20)
      },
      // 根据滚动的高度计算当前滚动到的索引，然后利用索引就能得到右侧的索引元素
      scrollY (newY) {
        // 滚动是从起始点顶点开始，而触摸滚动是从第一次触点算做起始点
        if (newY >= 0) {
          this.currentIndex = 0
          return
        }
        let listHeight = this.listHeight
        for (let i = 0; i < listHeight.length; i++) {
          let height1 = listHeight[i]
          let height2 = listHeight[i + 1]
          let y = -newY // 滚动的话，向下滚动都是负数，从0开始
          // 1. height2 有可能下标越界，获得undefined
          // 2. 在一个区间内
          if (!height2 || (y > height1 && y < height2)) {
            this.currentIndex = i
            console.log(i)
            return
          }
        }
        this.currentIndex = 0
      }
    }
  }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .list-view {
    width: 100%
    height: 100%
    overflow hidden //这个应该写在父级里面限制这个listview的把。没有明白为什么写在这里也有效果
    background $color-background
    .list-group {
      padding-bottom 30px
      .list-group-title {
        font-size $font-size-small
        line-height 30px
        padding-left 20px
        color $color-text-l
        background $color-highlight-background
      }
      .list-group-item {
        display flex
        align-items center
        padding 20px 0 0 30px
        .avtar {
          width: 50px
          height: 50px
          border-radius: 50%
        }
        .name {
          margin-left 20px
          color $color-text-l
          font-size $font-size-medium
        }
      }
    }
    .list-shortcut {
      position absolute
      right: 0
      top: 50%
      transform: translateY(-50%)
      width: 20px
      padding 20px 0
      background: $color-background-d
      border-radius: 10px
      text-align: center
      font-family: Helvetica
      .item {
        padding 3px
        color: $color-text-l
        font-size: $font-size-small
        line-height 1
        &.current {
          color: $color-theme
        }
      }
    }
  }
</style>

```

这个版本还有一个问题就是在手机端IOS10自带浏览器上，右侧定位会出现内容被截断显示不正常的问题。这个待以后测试；（这个问题调整了probeType又好了，不知道是为什么，派发滚动事件的时机很重要）
    






