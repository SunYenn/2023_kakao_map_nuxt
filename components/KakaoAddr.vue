<template>
  <div>
    <el-input
      v-model="addr"
      placeholder="주소를 입력해주세요"
      disabled
    >
      <el-button slot="append" icon="el-icon-search" @click="searchAddr" />
    </el-input>
  </div>
</template>
<script>
export default {
  name: 'KakaoMap',
  data () {
    return {
      addr: ''
    }
  },
  watch: {
    'addr' (val) {
      this.$emit('search-addr', val) // 부모 컴포넌츠 함수 실행
    }
  },
  mounted () {
    this.createScript()
  },
  methods: {
    createScript () {
      const scriptElement = document.createElement('script')
      scriptElement.src = '//t1.daumcdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js'
      scriptElement.type = 'text/javascript'
      document.head.appendChild(scriptElement)
    },
    searchAddr () {
      new window.daum.Postcode({ // 주소 검색
        oncomplete: (data) => {
          this.addr = data.roadAddress
        }
      }).open()
    }
  }
}
</script>
