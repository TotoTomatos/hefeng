<script setup>
import aioxs from 'axios'
import { ElMessage } from 'element-plus'
import { ref } from 'vue'

//获取地域id
const getLocationId = async (location) => {
  const res = await aioxs({
    url: `https://geoapi.qweather.com/v2/city/lookup`,
    method: 'get',
    params: {
      location,
      key: '51e86fa9caeb4509bd00bf486305905f'
    }
  })

  if (res.data.code == '200') {
    return res.data.location[0].id;
  } else {
    ElMessage.error('请输入正确的地名')
  }
}
//获取当前天气
const getNowWeather = async (locationId) => {
  const res = await aioxs({
    url: `https://devapi.qweather.com/v7/weather/now`,
    method: 'get',
    params: {
      location: locationId,
      key: '51e86fa9caeb4509bd00bf486305905f'
    }
  })
  if (res.data.code == '200') {
    return res.data.now
  } else {
    ElMessage.error('获取当前天气失败')
  }
}

//获取未来七天天气
const get7dWeather = async (locationId) => {
  const res = await aioxs({
    url: `https://devapi.qweather.com/v7/weather/7d`,
    method: 'get',
    params: {
      location: locationId,
      key: '51e86fa9caeb4509bd00bf486305905f'
    }
  })
  if (res.data.code == '200') {
    return res.data.daily
  } else {
    ElMessage.error('获取未来七天天气失败')
  }
}

const now = ref({})
const daily = ref([])
const nowLocationname = ref('')
const inputLocationName = ref('')

const getAllWeather = async (locationName) => {
  if (!locationName) {
    ElMessage.error('请输入地名')
    return
  }
  const locationId = await getLocationId(locationName)
  now.value = await getNowWeather(locationId)
  daily.value = await get7dWeather(locationId)
  nowLocationname.value = locationName
}

const formatDate = (dateString) => {
  const date = new Date(dateString)
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${month}月${day}日`
}

// 格式化带时区的日期为 yyyy-mm-dd
const formatDate2 = (dateString) => {
  const date = new Date(dateString)
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  return `${year}年${month}月${day}日`
}


getAllWeather('北京')


</script>

<template>
  <el-container>
    <el-header>
        <el-row justify="space-between" class="nav">
          <el-col :span="8">
            <span>{{ formatDate2(now.obsTime) }}</span>
          </el-col>
          <el-col :span="8">
            <el-icon size="18">
              <Location />
            </el-icon>
            <span>{{ nowLocationname }}</span>
            <el-input v-model="inputLocationName" style="max-width: 300px" placeholder="请输入">
              <template #append>
                <el-button type="primary" @click="getAllWeather(inputLocationName)" >搜索</el-button>
              </template>
            </el-input>
          </el-col>
        </el-row>
        <el-row justify="space-start" class="row2">
          <el-col :span="2">
            <span style="font-size: 90px;">
              {{ now.temp }}°
            </span>
          </el-col>
          <el-col :span="5" :offset="1">
            <el-row>
              <el-col style="font-size: 16px; ">
                <div v-if="now.feelsLike > 25" style="">体感温度：{{ now.feelsLike }}</div>
                <div v-if="now.feelsLike > 15 && now.feelsLike < 25">体感温度：{{ now.feelsLike }}</div>
                <div v-if="now.feelsLike < 15">体感温度：{{ now.feelsLike }}</div>
              </el-col>
            </el-row>
            <el-row>
              <el-col>
                天气：{{ now.text }}&nbsp;&nbsp; 风向：{{ now.windDir }} &nbsp;&nbsp;风力：{{ now.windScale }}级
              </el-col>
            </el-row>
          </el-col>
        </el-row>
        <el-row justify="space-start" class="row3">
          <el-col :span="4">
            能见度: {{ now.vis }} 公里
          </el-col>
          <el-col :span="4">
            大气压强: {{ now.pressure }} 百帕
          </el-col>
          <el-col :span="4">
            湿度: {{ now.humidity }}%
          </el-col>
          <el-col :span="4">
            风速: {{ now.windSpeed }}%
          </el-col>
        </el-row>

        <el-row justify="center" class="main">
          <el-col :span="18" style="background-color: #fff; border-radius: 5px;">
            <p style="font-size:20px; font-weight: 400;">七日内天气预报</p>
            <div class="cards">
              <el-card v-for="item in daily" shadow="never">
                <p>{{ formatDate(item.fxDate)}}</p>
                <img v-if="item.textNight == '晴'" src="https://hmajax.itheima.net/weather/qing.png" />
                <img v-if="item.textNight == '阴'" src="https://hmajax.itheima.net/weather/yin.png" />
                <img v-if="item.textNight == '多云'" src="https://hmajax.itheima.net/weather/duoyun.png" />
                <p style="line-height: 16px;">{{ item.textNight }}</p>
                <p style="line-height: 16px;">{{ item.tempMin }}~{{ item.tempMax }}℃</p>
                <span>{{ item.windDirDay }} </span><span>{{ item.windSpeedDay }}级</span>
              </el-card>
            </div>
          </el-col>
        </el-row>
    </el-header>
  </el-container>
</template>

<style scoped>
.read-the-docs {
  color: #fff;
}

.el-container {
  height: 100%;
  width: 100%;
}

.el-header {
  background-color: #409EFF;
  color: #fff;

}



.el-row {
  line-height: 50px;
}

.nav {
  margin-top: 30px;
  font-size: 18px;
  margin: 0 100px;
}

.row2 {

  margin: 50px 100px 0 100px;
  line-height: 90px;
}

.row3 {

  margin: 5px 100px 0 100px;
  line-height: 18px;
  font-size: 16px;
}

.el-input {
  border-radius: 50%;
}
.main{
  margin-top: 20px;
  color: black;
}
.cards {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
}
.el-card{
  flex: 1; /* 每个卡片均分宽度 */
  height:250px ;
  border-radius: 5%;
  border: 0;
}
.el-card :hover{
  background-color: #F7FAFF;
}

::v-deep .el-card__body {
 display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

}
</style>
