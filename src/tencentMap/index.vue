<template>
  <div id="tencent_map_show" v-show="visible">
    <div class="tencent_map">
      <div class="tencent_map_relative">
        <div class="tencent_map_modal clearfix">
          <span class="tencent_map_close" @click="handleClose">
            <i class="el-icon-circle-close"></i>
          </span>
          <div class="tencent_map_container">
            <div id="tencent_map_container"></div>
            <div></div>
          </div>
          <div class="tencent_map_search">
            <div class="tencent_map_city">
              <div class="margin_bottom_10">行政区</div>
              <!-- popper-append-to-body不能为true，会被全局遮挡 -->
              <el-select v-model="searchInfo.city" size="mini" filterable :popper-append-to-body="false" @change="handleCityChange">
                <el-option
                  v-for="item in cityList"
                  :key="item.id"
                  :label="item.fullname"
                  :value="item.fullname">
                </el-option>
              </el-select>
            </div>
            <div class="tencent_map_point margin_top_20">
              <div class="margin_bottom_10">地址</div>
              <el-select
                v-model="searchInfo.searchAddress"
                class="width_99"
                filterable
                remote
                placeholder="请输入地址"
                :remote-method="remoteMethod"
                :popper-append-to-body="false"
                @change="handleAddressChange"
                :loading="loading"
                size="mini"
              >
                <el-option
                  v-for="item in searchResultList"
                  :key="item.id"
                  :label="item.title"
                  :value="item.id">
                </el-option>
              </el-select>
            </div>
            <div class="tencent_map_point margin_top_20">
              <div class="margin_bottom_10">坐标</div>
              <el-input v-model="targetPoint.lat" class='width_49' size="mini" disabled></el-input>
              <el-input v-model="targetPoint.lng"  class='width_49 margin_left_1' size="mini" disabled></el-input>
            </div>
            <div class="tencent_map_address margin_top_20">
              <el-input type="textarea" class="width_99" v-model="targetPoint.address" size="mini"></el-input>
            </div>
            <div class="tencent_map_btn margin_top_20">
              <el-button type="primary" size="small" @click="handleClose">取消</el-button>
              <el-button type="primary" size="small" @click="handleSave">保存坐标</el-button>
            </div>
          </div>
        </div>
      </div>
    </div>
    
  </div>
</template>

<script>
import pjson from "./api/pjson";
import path from './api/path'
import cityList from './libs/city'
import pointSvg from './icon/point.svg'
export default {
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    mapcenter: {
      type: String,
      default: '39.90469,116.40717' //北京
    },
    mapKey: {
      type: String,
      require: true
    }
  },
  data() {
    return {
      searchInfo: {
        city: '北京市', //北京城市id
        searchAddress: ''
      },
      searchResultList: [],
      cityList,
      targetPoint: {
        address: '',
        lat: '',
        lng: ''
      },
      loading: false,
      pointSvg,
      markerPoint: null, //地图标记
      qqmap: null // 地图实例
    }
  },
  mounted() {
    if (typeof(qq) == 'object') {
      this.createMap();
    } else {
      this.loadTencentMap();
    }
    this.getCityList()
    window.onMapFileLoad = () => {
      console.log('qqmap加载完成')
      this.createMap();
    }
  },
  methods: {
    loadTencentMap() {
      let script = document.createElement("script");
      //设置标签的type属性
      script.type = "text/javascript";
      //设置标签的链接地址
      script.src = "https://map.qq.com/api/js?v=2.exp&callback=onMapFileLoad" + '&key=' + this.mapKey;
      //添加标签到dom
      document.body.appendChild(script);
    },
    //加载地图
    createMap() {
      let that = this;
      this.qqmap = new qq.maps.Map(document.getElementById("tencent_map_container"), {
        center: new qq.maps.LatLng(that.mapcenter.split(',')[0], that.mapcenter.split(',')[1]),      // 地图的中心地理坐标。
        zoom: 12,  // 地图的中心地理坐标。
        draggableCursor: 'crosshair'
      })
      this.$nextTick(() => {
        this.bindGetPoint()
        this.setMarker(39.90469,116.40717)
      })
    },
    // 获取城市列表
    getCityList() {
      // pjson(path.getCityList, {key: this.mapKey})
    },
    //关闭页面
    handleClose() {
      this.$emit('update:visible', false)
    },
    beforeClose() {
      //如果监听close，返回true,可以关闭，如果没有监听，默认返回vue，如果不想关闭弹窗请返回false
      const flag = this.$emit('close') 
      return flag
    },
    // 地址变化切换
    handleAddressChange(value) {
      this.searchResultList.forEach((item) => {
        if(item.id === value){
          this.targetPoint.address = item.address
          Object.assign(this.targetPoint, item.location)
          const { lat, lng } = this.targetPoint
          this.setMapCenter(lat, lng)
          this.setMapMarker(lat, lng)
        }
      })

    },
    handleSave() {
      // 输出信息
      let result = {...this.targetPoint}
      this.$emit('handleSave', result) 
      this.handleClose()
    },
    handleCityChange(value) {
      pjson(path.getCity, {key: this.mapKey, keyword: value})
      .then(res => {
        const { lat, lng } = res.result[0][0].location
        this.setMapCenter(lat, lng)
        this.setMarker(lat, lng)
      })
    },
    //获取城市列表接口设置中心点
    setMapCenter(lat, lng) {
      this.qqmap.setCenter(new qq.maps.LatLng(lat,lng));
    },
    //设置标记点
    setMapMarker(lat, lng) {
      this.markerPoint.setPosition(new qq.maps.LatLng(lat,lng));
    },
    //生成标记点
    setMarker(lat, lng) {
      this.markerPoint = new qq.maps.Marker({
          id: "marker-layer", //图层id
          map: this.qqmap,
          position: new qq.maps.LatLng(lat, lng),
          visible: true,
          icon: new qq.maps.MarkerImage(
            pointSvg,
            new qq.maps.Size(24, 24),
            new qq.maps.Point(0, 0),
            new qq.maps.Point(12, 24),
            new qq.maps.Size(24, 24)
          ),
      })
      qq.maps.event.addListener(this.markerPoint, 'click', (e) => {
        const { lat, lng } = e.latLng
        this.setMapCenter(lat, lng)
        this.qqmap.zoomTo(17);
      })
    },
    //用于查询地址数据
    remoteMethod(query) {
      const params = {
        key: this.mapKey,
        keyword: query.trim(),
        region: this.searchInfo.city
      }
      // adcode: 110105
      // address: "地铁15号线,地铁14号线东段"
      // category: "基础设施:交通设施:地铁站"
      // city: "北京市"
      // district: "朝阳区"
      // id: "10003975800530606021"
      // location: {lat: 39.998484, lng: 116.469364}
      // province: "北京市"
      // title: "望京[地铁站]"
      // type: 2
      if(params.keyword){
        this.loading = true
        pjson(path.suggestion, params)
        .then(res => {
          this.searchResultList = res.data || []
        })
        .finally(() => {
          this.loading = false
        })
      }
    },
    //定义事件处理方法
    bindGetPoint() {
      // cursorPixel: F {x: 374, y: 331.8333435058594}
      // latLng: p {lat: 39.88818221564929, lng: 116.40083312988281}
      // pixel: F {x: 374, y: 331.8333435058594}
      // _a: p {lat: 39.88818221564929, lng: 116.40083312988281}
      qq.maps.event.addListener(this.qqmap, 'click', (e) => {
        const {lat, lng} = e.latLng
        this.targetPoint.lat = lat
        this.targetPoint.lng = lng
        this.setMapMarker(lat, lng)
        pjson(path.geocode + `${lat},${lng}`, { key: this.mapKey })
        .then(res => {
          this.targetPoint.address = res.result.address
        })
        
      })
    }
  },
  watch: {
    visible(val, oldVal) {
      if(!val) { //关闭时
        const flag = this.beforeClose() 
        !flag && this.$emit('update:visible', true)
        Object.keys(this.targetPoint).forEach(key => {
          this.targetPoint[key] = ''
        })
        this.searchInfo = {
          city: '北京市', //北京城市id
          searchAddress: ''
        }
        this.setMapMarker(39.90469,116.40717)
        this.setMapCenter(39.90469,116.40717)
        this.qqmap.zoomTo(12)
      } 
    }
  }
};
</script>

<style lang="less" scoped>
.tencent_map{
  position: fixed;
  top: 0;
  left: 0;
  z-index: 3002;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
}
.tencent_map_relative{
  position: relative;
  width: 100%;
  height: 100%;
}
.tencent_map_container{
  float:left;
}
.tencent_map_modal{
  width: 1168px;
  height: 537px;
  position: absolute;
  background: #fff;
  box-shadow: 0 0 15px rgb(0 0 0 / 40%);
  border: 1px solid #ccc \ 9;
  left: 0;
  top: 0;
  right: 0;
  bottom: 0;
  margin: auto; 
  .tencent_map_close{
    display: block;
    cursor: pointer;
    width: 38px;
    height: 38px;
    overflow: hidden;
    position: absolute;
    right: -15px;
    top: -15px;
    outline: 0;
    z-index: 100;
    i{
      width: 38px;
      height: 38px;
      border-radius: 20px;
      font-size: 38px;
      background: black;
      color: white;
    }
    i:hover{
      color: rgb(235, 70, 41);
    }
  }
  #tencent_map_container{
    width: 783px;
    height: 537px;
  }
  .tencent_map_search{
    position: relative;
    width: 330px;
    padding: 18px 0 0 25px;
    height: 519px;
    overflow: hidden;
    float: left;
  }
  .margin_top_20{
    margin-top: 50px;
  }
  .width_49{
    width: 48%;
  }
  .width_99{
    width: 99%;
  }
  .margin_left_1{
    margin-left: 1%;
  }
  .margin_bottom_10{
    margin-bottom: 10px;
  }
}
.clearfix:after{
  content: '';
  height: 0;
  visibility: hidden;
  clear: both;
  display: block;
}

</style>
