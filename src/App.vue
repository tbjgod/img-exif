<template>
  <div id="app">
    <h1>上传照片获取数据</h1>
    <p>JavaScript 读取图像的原始数据的功能扩展，例如：拍照方向、相机设备型号、拍摄时间、ISO 感光度、GPS 地理位置等数据。</p>
    <div class="clearfix">
      <div class="left-side">
        <input type="file" @change="upload($event)" v-show="canUse"/>
        <br>
        <img :src="imgData" alt="image" id="img">
      </div>
      <div class="right-side">
        <textarea readonly placeholder="图片信息">{{imgShowInfo}}</textarea>
        <div id="map" v-show="locationInfo"></div>
      </div>
    </div>
  </div>
</template>

<script>
import EXIF from 'exif-js'
import axios from 'axios'

// 坐标系转换
function geoConv (coords) {
  console.log(coords)
  let convertor = new window.BMap.Convertor()
  let pointArr = []
  pointArr.push(new window.BMap.Point(coords.lng, coords.lat))
  convertor.translate(pointArr, 1, 5, data => {
    if (data.status === 0) {
      console.log(data.points)
      showMapFromGeo(data.points[0])
    }
  })
}
// 根据坐标显示地图
function showMapFromGeo (point) {
  // 添加地图
  let bm = new window.BMap.Map('map')
  bm.centerAndZoom(point, 15)
  bm.addControl(new window.BMap.NavigationControl())
  // 添加label
  let marker = new window.BMap.Marker(point)
  bm.addOverlay(marker)
  let label = new window.BMap.Label('照片拍摄地点', { offset: new window.BMap.Size(20, -10) })
  marker.setLabel(label)
}

export default {
  name: 'app',
  data () {
    return {
      canUse: true,
      imgData: '',
      imgInfo: '',
      imgShowInfo: '',
      locationInfo: false
    }
  },
  methods: {
    // 上传照片获取数据
    upload (e) {
      let file = e.target.files[0];
      if (file && !/image\/\w+/.test(file.type)) {
        alert("请确保文件为图像类型")
        return false
      }
      let reader = new FileReader()
      reader.readAsDataURL(file)
      this.getImgInfo(file)

      reader.onload = () => {
        this.imgData = reader.result
        e.target.value = ''
      }
    },
    // 检测浏览器是否支持
    checkFileReader () {
      if(typeof FileReader === 'undefined'){
        this.imgInfo = '抱歉，你的浏览器不支持 FileReader'
        this.canUse = false
      }
    },
    // 获取文件exif信息
    getImgInfo (img) {
      EXIF.getData(img, () => {
        this.imgInfo = EXIF.getAllTags(img)
        this.imgShowInfo = EXIF.pretty(img)
        this.getImgLocationInfo()
      })
    },
    // 获取文件所在地
    getImgLocationInfo () {
      if (this.imgInfo.GPSLongitude && this.imgInfo.GPSLatitude) {
        let lng = this.imgInfo.GPSLongitude[0] + this.imgInfo.GPSLongitude[1] / 60.0 + this.imgInfo.GPSLongitude[2] / 3600.0
        let lat = this.imgInfo.GPSLatitude[0] + this.imgInfo.GPSLatitude[1] / 60.0 + this.imgInfo.GPSLatitude[2] / 3600.0
        geoConv({lng, lat})
        this.locationInfo = true
      } else {
        this.locationInfo = false
      }
    }
  },
  mounted () {
    this.checkFileReader()
  }
}
</script>

<style lang="less">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

h1, h2 {
  font-weight: normal;
}

.clearfix {
  zoom: 1;
  &:after {
    content: '';
    display: block;
    clear: both;
  }
}

.left-side {
  float: left;
  width: 50%;

  img {
    margin-top: 10px;
    width: 90%;
    border: 1px solid black;
  }
}

.right-side {
  float: right;
  width: 50%;

  textarea {
    width: 90%;
    min-height: 200px;
    resize: none;
  }

  #map {
    margin: 10px auto 0;
    width: 90%;
    min-height: 300px;
  }
}
</style>
