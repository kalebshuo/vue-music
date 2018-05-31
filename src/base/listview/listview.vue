<template>
  <scroll :data="data" class="listview" ref="listview">
    <ul>
      <li v-for="(group,index) in data" class="list-group" ref="listGroup" :key="index">
        <h2 class="list-group-title">{{group.title}}</h2>
        <ul>
          <li @click="selectItem(item)" v-for="(item,index) in group.items" class="list-group-item" :key="index">
            <img class="avatar" v-lazy="item.avatar">
            <span class="name">{{item.name}}</span>
          </li>
        </ul>
      </li>
    </ul>
    <div class="list-shortcut" @touchstart="onShortcutTouchStart" @touchmove.stop.prevent="onShortcutTouchMove">
      <ul class="b"> <!-- 这里的touchstart touchmove是js的原生事件但是被better-scroll在内部重写了，原本的event.target事件指向的是最内层的dom（所以才会有事件代理）。.stop.prevent是阻止冒泡和默认事件（这个是vue提供的）-->
        <li v-for="(item, index) in shortcutList" class="item"
        :key="index" :data-index="index">
          {{item}}
          <span class="a"></span>
        </li>
      </ul>
    </div>
  </scroll>
</template>

<script type='text/ecmascript-6'>
import Scroll from 'base/scroll/scroll'
import {getData} from 'common/js/dom'

const ANCHOR_HEIGHT = 18

export default {
  created() {
    this.touch = {}
  },
  props: {
    data: {
      type: Array,
      default: null
    }
  },
  components: {
    Scroll
  },
  computed: {
    shortcutList() {
      return this.data.map((group) => { // 数组的每一项是一个对象，每个对象的title按A-Z的顺序排列
        return group.title.substr(0, 1)
      })
    }
  },
  methods: {
    onShortcutTouchStart(e) {
      // console.log(e) // touchevent是一个对象，里面包含着点击的dom对象
      // console.log(e.target)
      let anchorIndex = getData(e.target, 'index')
      let firstTouch = e.touches[0]
      this.touch.y1 = firstTouch.pageY
      this.touch.anchorIndex = anchorIndex
      this._scrollTo(anchorIndex);
    },
    onShortcutTouchMove(e) {
      let firstTouch = e.touches[0]
      // console.log(e.touches)
      // console.log(firstTouch)
      // console.log(firstTouch.pageY)
      this.touch.y2 = firstTouch.pageY
      let delta = (this.touch.y2 - this.touch.y1) / ANCHOR_HEIGHT | 0
      // 除以每一个字母的高度（18px）然后向下取整，得到滑动了几个字母块
      let anchorIndex = parseInt(this.touch.anchorIndex) + delta // getData()返回的是字符串，所以要转换成int类型再去相加
      this._scrollTo(anchorIndex)
      },
    _scrollTo(index) {
      this.$refs.listview.scrollToElement(this.$refs.listGroup[index],0)
    }
  }
};
</script>

<style lang='stylus' rel='stylesheet/stylus'>
  @import "~common/stylus/variable"

  .listview
    position: relative
    width: 100%
    height: 100%
    overflow: hidden
    background: $color-background
    .list-group
      padding-bottom: 30px
      .list-group-title
        height: 30px
        line-height: 30px
        padding-left: 20px
        font-size: $font-size-small
        color: $color-text-l
        background: $color-highlight-background
      .list-group-item
        display: flex
        align-items: center
        padding: 20px 0 0 30px
        .avatar
          width: 50px
          height: 50px
          border-radius: 50%
        .name
          margin-left: 20px
          color: $color-text-l
          font-size: $font-size-medium
    .list-shortcut
      position: absolute // 首屏位置是不变的，变的是第一个子元素的transitionY，详情见better-scroll原理
      z-index: 30
      right: 0
      top: 50%
      transform: translateY(-50%)
      width: 20px
      padding: 20px 0
      border-radius: 10px
      text-align: center
      background: $color-background-d
      font-family: Helvetica
      .item
        padding: 3px
        line-height: 1
        color: $color-text-l
        font-size: $font-size-small
        &.current
          color: $color-theme
    .list-fixed
      position: absolute
      top: 0
      left: 0
      width: 100%
      .fixed-title
        height: 30px
        line-height: 30px
        padding-left: 20px
        font-size: $font-size-small
        color: $color-text-l
        background: $color-highlight-background
    .loading-container
      position: absolute
      width: 100%
      top: 50%
      transform: translateY(-50%)
</style>