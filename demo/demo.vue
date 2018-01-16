<template>
  <div class="demo">
    <p>pull direction: down</p>
    <pull-to-refresh
      :style="{ height: '250px', overflow: 'auto', border: '1px solid #ccc' }"
      className="forTest"
      :refreshing="false"
      :indicator="{deactivate: 'pull down'}"
    >
      <div v-for="item in list" :style="{ textAlign: 'center', padding: '20px' }" :key="item">{{ item }}</div>
    </pull-to-refresh>
    <p>pull direction: up</p>
    <pull-to-refresh
      :style="{ height: '250px', overflow: 'auto', border: '1px solid #ccc' }"
      direction="up"
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
      list: [1, 2, 3, 4, 5]
    }
  },
  components: {
    PullToRefresh
  }
}
</script>

