<template>
  <div id="app">
    <TencentMap :visible.sync="visible" />
    <el-button @click="handleShow">打开地图弹窗</el-button>
  </div>
</template>

<script>
import TencentMap from './tencentMap'

export default {
  name: 'App',
  components: {
    TencentMap
  },
  data() {
    return {
      visible: false,
      mapKey: ''
    }
  },
  methods: {
    handleShow() {
      this.visible = true
    }
  }
}
</script>

<style lang="less">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
