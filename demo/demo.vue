<template>
  <div class="demo">
    <pull-to-refresh
      :style="{ height: '300px', overflow: 'auto', border: '1px solid #ccc' }"
      className="forTest"
      :refreshing="false"
      :indicator="{deactivate: '下拉'}"
    >
      <div v-for="item in list" :style="{ textAlign: 'center', padding: '20px' }" :key="item">{{ item }}</div>
    </pull-to-refresh>
  </div>
</template>

<script>
import PullToRefresh from '../src/pulltorefresh';
export default {
  data () {
    return {
      list: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    }
  },
  methods: {
    getBody () {
      return document.body
    },
    refresh () {

    },
    getBody () {
      return document.body;
    }
  },
  components: {
    PullToRefresh
  }
}
</script>

