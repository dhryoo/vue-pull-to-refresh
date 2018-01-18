# v-pull-to-refresh

Vue PullToRefresh Component.

## Install

```shell
npm install --save v-pull-to-refresh
```

## Usage

```javascript
import PullToRefresh from 'v-pull-to-refresh';

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
```

```html
<pull-to-refresh
  :style="{ height: '250px', overflow: 'auto', border: '1px solid #ccc' }"
  className="forTest"
  :refreshing="false"
  :indicator="{deactivate: 'pull down'}"
>
  <div v-for="item in list" :style="{ textAlign: 'center', padding: '20px' }" :key="item">{{ item }}</div>
</pull-to-refresh>
```

## API

### Props

| name     | description    | type     | default      |
|----------|----------------|----------|--------------|
| direction  | pull direction, can be `up` or `down` | String | `down` |
| distanceToRefresh | distance to pull to refresh | number | 50  |
| refreshing | Whether the view should be indicating an active refresh | bool | false |
| onRefresh  | Called when the view starts refreshing. | () => void | - |
| indicator  | indicator config | Object | `{ activate: 'release', deactivate: 'pull', release: 'loading', finish: 'finish' }` |
| className | additional css class of root dom node | String | - |
| prefixCls | prefix class | String | 'v-pull-to-refresh' |