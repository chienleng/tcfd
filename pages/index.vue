<template>
  <section>
    <Map
      :data="d"
      :show="show"
    />
    <div class="concept-box">
      <div>Data from 
        <a 
          href="https://gist.github.com/chienleng/119fbb758be86317c894647b7caec264"
          target="_blank">github gist
        </a>
      </div>
      <div><strong>{{ num }}</strong> points</div>
      <div>
        <label>
          <input 
            v-model="show"
            type="checkbox">
          Show datapoints
        </label>
      </div>
    </div>
  </section>
</template>

<script>
import axios from 'axios'
import Map from '~/components/Map.vue'

export default {
  components: {
    Map
  },

  data() {
    return {
      d: [],
      num: 0,
      show: true
    }
  },

  mounted() {
    const url =
      'https://gist.githubusercontent.com/chienleng/119fbb758be86317c894647b7caec264/raw/c6b1c1753aa985a238412104794a18b626e1798f/tc_exposure.json'
    axios.get(url).then(res => {
      const features = res.data.features
      this.d = features
      this.num = features.length
    })
  }
  // asyncData({ $axios }) {
  //   return $axios.get('/data.json').then(res => {
  //     const features = res.data.features
  //     return {
  //       d: features,
  //       num: features.length
  //     }
  //   })
  // }
}
</script>

<style>
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Helvetica Neue', sans-serif;
}
.concept-box {
  position: absolute;
  top: 30px;
  right: 30px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 2px;
  padding: 5px 10px;
  font-size: 12px;
}
</style>
