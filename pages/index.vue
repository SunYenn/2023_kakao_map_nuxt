<template>
  <div style="text-align: center; padding: 30px;">
    <h2>카카오 맵</h2>
    <div class="mapViewBox">
      <KakaoAddr @search-addr="MapCallbyAddr" />
      <KakaoMap ref="mapCpnt" />
    </div>
  </div>
</template>
<script>
import KakaoMap from '~/components/KakaoMap.vue'
import KakaoAddr from '~/components/KakaoAddr.vue'
export default {
  name: 'MapView',
  components: { KakaoMap, KakaoAddr },
  methods: {
    MapCallbyAddr (data) {
      this.$refs.mapCpnt.searchPlace(data)
    }
  }
}
</script>
<style scoped>
.mapViewBox {
  width: 500px;
  margin: 10px auto;
}
</style>
