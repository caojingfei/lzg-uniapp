<template>
  <view
    :class="['uni-collapse-cell',{'uni-collapse-cell--disabled':disabled,'uni-collapse-cell--open':isOpen}]"
    :hover-class="disabled ? '' : 'uni-collapse-cell--hover'">
    <view
      class="uni-collapse-cell__title"
      @click="onClick">
      <view
        v-if="thumb"
        class="uni-collapse-cell__title-extra">
        <image
          :src="thumb"
          class="uni-collapse-cell__title-img"/>
      </view>
      <view class="uni-collapse-cell__title-inner">
        <view class="uni-collapse-cell__title-text">{{ title }}</view>
      </view>
      <view
        :class="{'uni-active':isOpen,'uni-collapse-cell--animation':showAnimation===true}"
        class="uni-collapse-cell__title-arrow">
        <uni-icons
          color="#bbb"
          size="20"
          type="arrowdown"/>
      </view>
    </view>
    <view
      :class="{'uni-collapse-cell--animation':showAnimation===true}"
      :style="{height:isOpen ? height : '0px'}"
      class="uni-collapse-cell__content">
      <view :id="elId">
        <slot />
      </view>
    </view>
  </view>
</template>

<script>
import { uniIcons } from "@dcloudio/uni-ui";

export default {
  name: 'UniCollapseItem',
  components: {
    uniIcons
  },
  props: {
    title: { // 列表标题
      type: String,
      default: ''
    },
    name: { // 唯一标识符
      type: [Number, String],
      default: 0
    },
    disabled: { // 是否禁用
      type: Boolean,
      default: false
    },
    showAnimation: { // 是否显示动画
      type: Boolean,
      default: false
    },
    open: { // 是否展开
      type: Boolean,
      default: false
    },
    thumb: { // 缩略图
      type: String,
      default: ''
    }
  },
  data () {
    /**
     * TODO 兼容新旧编译器
     * 新编译器（自定义组件模式）下必须使用固定数值，否则部分平台下会获取不到节点。
     * 随机数值是在旧编译器下使用的，旧编译器模式已经不推荐使用，后续直接废掉随机数值的写法。
     */
    const elId = this.__callback_hook ? 'uni_collapse_item' : `Uni_${Math.ceil(Math.random() * 10e5).toString(36)}`
    return {
      isOpen: false,
      height: 'auto',
      elId: elId
    }
  },
  watch: {
    open (val) {
      this.open = val
    }
  },
  inject: ['collapse'],
  created () {
    this.isOpen = this.open
    this.nameSync = this.name ? this.name : this.collapse.childrens.length
    this.collapse.childrens.push(this)
    if (this.collapse.accordion) {
      if (this.isOpen) {
        let lastEl = this.collapse.childrens[this.collapse.childrens.length - 2]
        if (lastEl) {
          this.collapse.childrens[this.collapse.childrens.length - 2].isOpen = false
        }
      }
    }
  },
  // #ifdef H5
  mounted () {
    this.getSize()
  },
  // #endif
  // #ifndef H5
  onReady () {
    this.getSize()
  },
  // #endif
  methods: {
    getSize () {
      if (this.showAnimation) {
        uni.createSelectorQuery().in(this).select(`#${this.elId}`).boundingClientRect().exec((ret) => {
          this.height = ret[0].height + 'px'
        })
      }
    },
    onClick () {
      if (this.disabled) {
        return
      }
      this.collapse.childrens.forEach(vm => {
        if (vm === this) {
          this.isOpen = !this.isOpen
          return
        }
        vm.isOpen = false
      })
      this.collapse.onChange && this.collapse.onChange()
    }
  }
}
</script>

<style lang="scss">
	@mixin collapse-hover {
		background-color: $uni-bg-color-hover;
	}

	@mixin collapse-disabled {
		opacity: 0.3;
	}

	$collapse-title-pd:$uni-spacing-col-lg $uni-spacing-row-lg;

	.uni-collapse-cell {
		position: relative;

		&--hover {
			@include collapse-hover;
		}

		&--open {
			@include collapse-hover;
		}

		&--disabled {
			@include collapse-disabled;
		}

		&--animation {
			transition: all 0.3s;
		}

		&:after {
			position: absolute;
			z-index: 3;
			right: 0;
			bottom: 0;
			left: 0px;
			height: 1px;
			content: '';
			-webkit-transform: scaleY(0.5);
			transform: scaleY(0.5);
			background-color: transparent;
		}

		&__title {
			padding-left: 55rpx;
      padding-right: 25rpx;
			width: 100%;
      height: 88rpx;
      line-height: 88rpx;
			box-sizing: border-box;
      background: #eee;
			flex: 1;
			position: relative;
			display: flex;
			flex-direction: row;
			justify-content: space-between;
			align-items: center;

			&-extra {
				margin-right: 18upx;
				display: flex;
				flex-direction: row;
				justify-content: center;
				align-items: center;
			}

			&-img {
				height: $uni-img-size-base;
				width: $uni-img-size-base;
			}

			&-arrow {
				width: 20px;
				height: 88rpx;
        line-height: 88rpx;
				transform: rotate(0deg);
				transform-origin: center center;

				&.uni-active {
					transform: rotate(-180deg);
				}
			}

			&-inner {
				flex: 1;
				overflow: hidden;
				display: flex;
				flex-direction: column;
			}

			&-text {
				color: #333;
        font-size: 30rpx;
				text-overflow: ellipsis;
				white-space: nowrap;
				color: inherit;
				line-height: 1.5;
				overflow: hidden;
			}
		}

		&__content {
			position: relative;
			width: 100%;
			overflow: hidden;
			background: $uni-bg-color;

			view {
				font-size: $uni-font-size-base;
			}
		}
	}
</style>
