<template>
  <div v-if="getScrollContainer()" :class="`${prefixCls}-content-wrapper`">
    <div ref="content" :class="contentCls">
      <slot v-if="direction === 'up'"></slot>
      <div :class="`${prefixCls}-indicator`">{{ indicator[currSt] || INDICATOR[currSt] }}</div>
      <slot v-if="direction === 'down'"></slot>
    </div>
  </div>
  <div v-else ref="container" :class="containerCls">
    <div :class="`${prefixCls}-content-wrapper`">
      <div ref="content" :class="contentCls">
        <slot v-if="direction === 'up'"></slot>
        <div :class="`${prefixCls}-indicator`">{{ indicator[currSt] || INDICATOR[currSt] }}</div>
        <slot v-if="direction === 'down'"></slot>
      </div>
    </div>
  </div>
</template>

<script>
import classnames from 'classnames';

const isWebView = typeof navigator !== 'undefined' && /(iPhone|iPod|iPad).*AppleWebKit(?!.*Safari)/i.test(navigator.userAgent);
const DOWN = 'down';
const UP = 'up';
const INDICATOR = { 
  activate: 'release', 
  deactivate: 'pull', 
  release: 'loading', 
  finish: 'finish'
};

let supportsPassive = false;
try {
  const opts = Object.defineProperty({}, 'passive', {
    get () {
      supportsPassive = true;
    },
  });
  window.addEventListener('test', opts, opts);
  window.removeEventListener('test', opts, opts);
} catch (e) {
  supportsPassive = false;
}

const willPreventDefault = supportsPassive ? { passive: false } : false;

export default {
  name: 'VPullToRefresh',
  data () {
    return {
      containerRef: null,
      contentRef: null,
      _to: null,
      _ScreenY: null,
      _startScreenY: null,
      _lastScreenY: null,
      _timer: undefined,
      currSt: '',
      dragOnEdge: false,
      refreshingState: this.refreshing,
      INDICATOR
    }
  },
  props: {
    prefixCls: {
      type: String,
      default: 'v-pull-to-refresh'
    },
    direction: {
      type: String,
      default: DOWN
    },
    distanceToRefresh: {
      type: Number,
      default: 25
    },
    refreshing: false,
    indicator: {
      type: Object,
      default () {
        return {}
      }
    },
    className: String,
    onRefresh: Function,
    getScrollContainer: {
      type: Function,
      default: () => undefined
    }
  },
  computed: {
    containerCls () {
      return classnames(this.className, this.prefixCls, `${this.prefixCls}-${this.direction}`)
    },
    contentCls () {
      if (this.getScrollContainer()) {
        return classnames(`${this.prefixCls}-content`, `${this.prefixCls}-${this.direction}`, !this.dragOnEdge && `${this.prefixCls}-transition`)
      } else {
        return classnames(`${this.prefixCls}-content`, !this.dragOnEdge && `${this.prefixCls}-transition`)
      }
    }
  },
  methods: {
    init (ele) {
      if (!ele) {
        return;
      }
      this._to = {
        touchstart: this.onTouchStart.bind(this, ele),
        touchmove: this.onTouchMove.bind(this, ele),
        touchend: this.onTouchEnd.bind(this, ele),
        touchcancel: this.onTouchEnd.bind(this, ele)
      };
      Object.keys(this._to).forEach(key => {
        ele.addEventListener(key, this._to[key], willPreventDefault);
      })
    },
    destroy (ele) {
      if (!this._to || !ele) {
        return;
      }
      Object.keys(this._to).forEach(key => {
        ele.removeEventListener(key, this._to[key]);
      });
    },
    setTransform (nodeStyle, value) {
      nodeStyle.transform = value;
      nodeStyle.webkitTransform = value;
      nodeStyle.MozTransform = value;
    },
    setContentStyle (ty) {
      if (this.contentRef) {
        this.setTransform(this.contentRef.style, `translate3d(0px,${ty}px,0)`);
      }
    },
    isEdge (ele, direction) {
      const container = this.getScrollContainer();
      if (container && container === document.body) {
        const scrollNode = document.scrollingElement ? document.scrollingElement : document.body;
        if (direction === UP) {
          return scrollNode.scrollHeight - scrollNode.scrollTop <= window.innerHeight;
        }
        if (direction === DOWN) {
          return scrollNode.scrollTop <= 0;
        }
      }
      if (direction === UP) {
        return ele.scrollHeight - ele.scrollTop === ele.clientHeight;
      }
      if (direction === DOWN) {
        return ele.scrollTop <= 0;
      }
    },
    onTouchStart (ele, e) {
      this._ScreenY = this._startScreenY = e.touches[0].screenY;
      this._lastScreenY = this._lastScreenY || 0;
    },
    onTouchMove (ele, e) {
      const _screenY = e.touches[0].screenY;
      const direction = this.direction;

      if (direction === UP && this._startScreenY < _screenY || 
        direction === DOWN && this._startScreenY > _screenY) {
        return;
      }

      if (this.isEdge(ele, direction)) {
        if (!this.dragOnEdge) {
          this.dragOnEdge = true;
        }
        e.preventDefault();

        const _diff = Math.round(_screenY - this._ScreenY);
        this._ScreenY = _screenY;
        this._lastScreenY += _diff;

        this.setContentStyle(this._lastScreenY);

        if (Math.abs(this._lastScreenY) < this.distanceToRefresh) {
          if (this.currSt !== 'deactivate') {
            this.currSt = 'deactivate';
          }
        } else {
          if (this.currSt === 'deactivate') {
            this.currSt = 'activate';
          }
        }

        if (isWebView && e.changedTouches[0].clientY < 0) {
          this.onTouchEnd();
        }
      }
    },
    onTouchEnd (ele, e) {
      if (this.dragOnEdge) {
        this.dragOnEdge = false;
      }
      if (this.currSt === 'activate') {
        this.currSt = 'release';
        this._timer = setTimeout(() => {
          if (!this.refreshingState) {
            this.currSt = 'finish';
            this.reset();
          }
          this._timer = undefined;
        }, 1000);
        
        if (typeof this.onRefresh === 'function') {
          this.onRefresh()
        } else {
          this.refreshAction();
        }        
      } else {
        this.reset();
      }
    },
    refreshAction () {
      this.refreshingState = true;
      setTimeout(() => {
        this.refreshingState = false;
      }, 1000);
    },
    reset () {
      this._lastScreenY = 0;
      this.setContentStyle(0);
    },
    triggerPullToRefresh () {
      if (!this.dragOnEdge) {
        if (this.refreshingState) {
          if (this.direction === UP) {
            this._lastScreenY = - this.distanceToRefresh - 1;
          }
          if (this.direction === DOWN) {
            this._lastScreenY = this.distanceToRefresh + 1;
          }
          this.currSt = 'release';
          this.setContentStyle(this._lastScreenY);
        } else {
          this.currSt = 'finish';
          this.reset();
        }
      }
    }
  },
  mounted () {
    this.containerRef = this.$refs.container;
    this.contentRef = this.$refs.content;
    
    this.init(this.getScrollContainer() || this.containerRef);
    this.triggerPullToRefresh();
  },
  destroyed () {
    this.destroy(this.getScrollContainer() || this.containerRef);
  },
  watch: {
    refreshingState (n, o) {
      this.triggerPullToRefresh();
    }
  }
}
</script>

<style>
.v-pull-to-refresh-content {
  transform-origin: left top 0px;
}
.v-pull-to-refresh-content-wrapper {
  overflow: hidden;
}
.v-pull-to-refresh-transition {
  transition: transform 0.3s;
}
.v-pull-to-refresh-indicator {
  color: grey;
  text-align: center;
  height: 25px;
  line-height: 25px;
}
.v-pull-to-refresh-down .v-pull-to-refresh-indicator {
  margin-top: -25px;
}
.v-pull-to-refresh-up .v-pull-to-refresh-indicator {
  margin-bottom: -25px;
}
</style>


