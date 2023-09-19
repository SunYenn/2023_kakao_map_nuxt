<template>
  <div class="mapContainer">
    <script type="text/javascript" :src="loadScript()"></script>
    <div ref="map" class="map" />
  </div>
</template>

<script>
export default {
  name: 'KakaoMap', // 컴포넌트 이름 지정
  data () {
    return {
      map: null,
      mapOption: {
        level: 5,
        x: 126.8980457,
        y: 37.5203274
      }
    }
  },
  watch: {
    'mapOption.x' (val) {
      this.initMap()
    },
    'mapOption.y' (val) {
      this.initMap()
    }
  },
  created () {
  },
  mounted () {
    // api 스크립트 소스 불러오기 및 지도 출력
    if (window.kakao) {
      this.initMap()
      this.getCurrLoc()
    } else {
      this.loadScript()
    }
  },
  methods: {
    // api 불러오기
    loadScript () {
      return '//dapi.kakao.com/v2/maps/sdk.js?appkey=' + process.env.VUE_APP_KAKAO_JS_KEY + '&libraries=services,clusterer,drawing'
    },
    // 맵 초기화
    initMap () {
      const container = this.$refs.map
      const options = {
        center: new window.kakao.maps.LatLng(this.mapOption.y, this.mapOption.x),
        level: this.mapOption.level
      }

      this.map = new window.kakao.maps.Map(container, options) // 지도 생성

      // 지도에 지형정보를 표시하도록 지도타입 추가
      this.map.addOverlayMapTypeId(window.kakao.maps.MapTypeId.TERRAIN)

      // 일반 지도와 스카이뷰로 지도 타입을 전환할 수 있는 지도타입 컨트롤 생성
      const mapTypeControl = new window.kakao.maps.MapTypeControl()
      this.map.addControl(mapTypeControl, window.kakao.maps.ControlPosition.TOPRIGHT)

      // 지도 확대 축소를 제어할 수 있는 줌 컨트롤 생성
      const zoomControl = new window.kakao.maps.ZoomControl()
      this.map.addControl(zoomControl, window.kakao.maps.ControlPosition.RIGHT)

      // marker 생성
      const markerPosition = new window.kakao.maps.LatLng(this.mapOption.y, this.mapOption.x)
      const marker = new window.kakao.maps.Marker({
        position: markerPosition
      })
      marker.setMap(this.map)

      // 클릭 이벤트 생성
      window.kakao.maps.event.addListener(this.map, 'click', (mouseEvent) => {
        console.log('지도에서 클릭한 위치의 좌표는 ' + mouseEvent.latLng.toString() + ' 입니다.')
        const level = this.map.getLevel()
        console.log('현재 지도 level : ' + level)
      })

      // 이동 이벤트 생성
      window.kakao.maps.event.addListener(this.map, 'center_changed', () => {
        const latlng = this.map.getCenter()
        console.log('현재 지도의 중심 좌표는 ' + latlng.toString() + ' 입니다.')
      })
    },
    // GPS 정보 가져오기
    getCurrLoc () {
      const self = this
      if (navigator.geolocation) {
        // GeoLocation을 이용해서 접속 위치를 얻어옵니다
        navigator.geolocation.getCurrentPosition(function (position) {
          self.mapOption.y = position.coords.latitude // 위도
          self.mapOption.x = position.coords.longitude // 경도
        })
      }
    },
    // 검색 이벤트
    searchPlace (data) {
      const self = this

      // 지도 중심 이동
      const moveLatLon = new window.kakao.maps.LatLng(self.mapOption.y, self.mapOption.x)
      this.map.panTo(moveLatLon)

      const geocoder = new window.kakao.maps.services.Geocoder()
      geocoder.addressSearch(data, function (result, status) {
        // 정상적으로 검색이 완료됐으면
        if (status === window.kakao.maps.services.Status.OK) {
          self.mapOption.x = result[0].x
          self.mapOption.y = result[0].y

          self.$axios.post('/api/map/getCoords', self.mapOption)
            .then((response) => {
              self.searchMark(response.data)
            })
            .catch((error) => {
              console.error(error)
            })
        } else {
          self.$message.error('검색 결과가 존재하지 않습니다.')
        }
      })
    },
    // 검색 결과 marking
    searchMark (data) {
      // data = [
      //   { locName: '서울신천초등학교', x: 127.0950476, y: 37.51398996 },
      //   { locName: '잠실중학교', x: 127.0993332, y: 37.51733281 },
      //                            ...
      // ]
      data.forEach((item) => {
        const imageSrc = 'https://i1.daumcdn.net/dmaps/apis/n_local_blit_04.png' // 마커이미지 주소
        const imageSize = new window.kakao.maps.Size(25, 33) // 마커이미지의 크기

        // 마커의 이미지정보를 가지고 있는 마커이미지 생성
        const markerImage = new window.kakao.maps.MarkerImage(imageSrc, imageSize)

        // 마커 생성
        const marker = new window.kakao.maps.Marker({
          map: this.map,
          position: new window.kakao.maps.LatLng(item.y, item.x),
          image: markerImage
        })

        // 인포윈도우 생성
        const infowindow = new window.kakao.maps.InfoWindow({
          content: `<span>${item.locName}</span>`
        })

        // 마커에 mouseover 이벤트 등록
        window.kakao.maps.event.addListener(marker, 'mouseover', () => {
          infowindow.open(this.map, marker)
        })

        // 마커에 mouseout 이벤트 등록
        window.kakao.maps.event.addListener(marker, 'mouseout', () => {
          infowindow.close()
        })
      })
    }
  }
}
</script>
<style scoped>
.mapContainer {
  text-align: center;
  width: 500px;
  height: 470px;
  box-sizing: border-box;
}

.map {
  text-align: center;
  height: calc(100% - 70px);
}
</style>
