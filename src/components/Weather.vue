<template>
  <div class="weather" v-if="weatherData.adCode.city && weatherData.weather.weather">
    <div :class="{ 'weather-two-lines': shouldSplitLines }">
      <template v-if="shouldSplitLines">
        <!-- 超长时第一行 - 只显示地区 -->
        <div class="weather-line">
          <span v-if="!mainKey">{{ weatherData.adCode.region }}&nbsp;</span>
          <span>{{ weatherData.adCode.city }}</span>
        </div>
        
        <!-- 超长时第二行 - 显示天气、温度和风向信息 -->
        <div class="weather-line">
          <span>{{ weatherData.weather.weather }}&nbsp;</span>
          <span>{{ weatherData.weather.temperature }}℃</span>
          <template v-if="mainKey">
            <span class="sm-hidden">
              &nbsp;{{
                weatherData.weather.winddirection?.endsWith("风")
                  ? weatherData.weather.winddirection
                  : weatherData.weather.winddirection + "风"
              }}&nbsp;
            </span>
            <span class="sm-hidden">{{ weatherData.weather.windpower }}&nbsp;级</span>
          </template>
        </div>
      </template>

      <!-- 未超长时单行显示 -->
      <template v-else>
        <div class="weather-line">
          <span v-if="!mainKey">{{ weatherData.adCode.region }}&nbsp;</span>
          <span>{{ weatherData.adCode.city }}&nbsp;</span>
          <span>{{ weatherData.weather.weather }}&nbsp;</span>
          <span>{{ weatherData.weather.temperature }}℃</span>
          <template v-if="mainKey">
            <span class="sm-hidden">
              &nbsp;{{
                weatherData.weather.winddirection?.endsWith("风")
                  ? weatherData.weather.winddirection
                  : weatherData.weather.winddirection + "风"
              }}&nbsp;
            </span>
            <span class="sm-hidden">{{ weatherData.weather.windpower }}&nbsp;级</span>
          </template>
        </div>
      </template>
    </div>
  </div>
  <div class="weather" v-else>
    <span>天气数据获取失败</span>
  </div>
</template>

<script>
export default {
  computed: {
    fullText() {
      let text = '';
      if (!this.mainKey) {
        text += this.weatherData.adCode.region + ' ';
      }
      text += this.weatherData.adCode.city + ' ' +
              this.weatherData.weather.weather + ' ' +
              this.weatherData.weather.temperature + '℃';
      
      if (this.mainKey) {
        text += ' ' + 
               (this.weatherData.weather.winddirection?.endsWith("风")
                  ? this.weatherData.weather.winddirection
                  : this.weatherData.weather.winddirection + "风") +
               ' ' + this.weatherData.weather.windpower + ' 级';
      }
      return text;
    },
    shouldSplitLines() {
      return this.fullText.length > 240;
    }
  }
}
</script>

<style scoped>
.weather-two-lines {
  display: flex;
  flex-direction: column;
  gap: 4px; /* 两行之间的间距 */
}

.weather-line {
  display: flex;
  align-items: center;
}
</style>


<!--
<template>
  <div class="weather" v-if="weatherData.adCode.city && weatherData.weather.weather">
    <span v-if="!mainKey">{{ weatherData.adCode.region }}&nbsp;</span>
    <span>{{ weatherData.adCode.city }}&nbsp;</span>
    <span>{{ weatherData.weather.weather }}&nbsp;</span>
    <span>{{ weatherData.weather.temperature }}℃</span>
<!--
    <span class="sm-hidden">
      &nbsp;{{
        weatherData.weather.winddirection?.endsWith("风")
          ? weatherData.weather.winddirection
          : weatherData.weather.winddirection + "风"
      }}&nbsp;
    </span>
    <span class="sm-hidden">{{ weatherData.weather.windpower }}&nbsp;级</span>
<!--
    <template v-if="mainKey">
    <span class="sm-hidden">
        &nbsp;{{
            weatherData.weather.winddirection?.endsWith("风")
            ? weatherData.weather.winddirection
            : weatherData.weather.winddirection + "风"
        }}&nbsp;
    </span>
    <span class="sm-hidden">{{ weatherData.weather.windpower }}&nbsp;级</span>
    </template>
  </div>
  <div class="weather" v-else>
    <span>天气数据获取失败</span>
  </div>
</template>
-->
<script setup>
import { getAdcode, getWeather, getOtherWeather } from "@/api";
import { Error } from "@icon-park/vue-next";

// 高德开发者 Key
const mainKey = import.meta.env.VITE_WEATHER_KEY;

// 天气数据
const weatherData = reactive({
  adCode: {
    city: null, // 城市
    adcode: null, // 城市编码
  },
  weather: {
    weather: null, // 天气现象
    temperature: null, // 实时气温
    winddirection: null, // 风向描述
    windpower: null, // 风力级别
  },
});

// 取出天气平均值
const getTemperature = (min, max) => {
  try {
    // 计算平均值并四舍五入
    const average = (Number(min) + Number(max)) / 2;
    return Math.round(average);
  } catch (error) {
    console.error("计算温度出现错误：", error);
    return "NaN";
  }
};

// 获取天气数据
const getWeatherData = async () => {
  try {
    // 获取地理位置信息
    if (!mainKey) {
      
/*
      console.log("未配置，使用备用天气接口");
      const result = await getOtherWeather();
      console.log(result);
      const data = result.result;
      weatherData.adCode = {
        city: data.city.City || "未知地区",
        // adcode: data.city.cityId,
      };
      weatherData.weather = {
        weather: data.condition.day_weather,
        temperature: getTemperature(data.condition.min_degree, data.condition.max_degree),
        winddirection: data.condition.day_wind_direction,
        windpower: data.condition.day_wind_power,
      };
*/

      console.log("未配置，使用备用天气接口");
      const result = await getOtherWeather();
      console.log(result);
      const data = result;  // 不需要 .result 了，因为数据在顶层
      
      weatherData.adCode = {
          city: data.location?.city || "未知地区",
          // 如果需要 region 也可以加上
          region: data.location?.region
      };
      
      weatherData.weather = {
          weather: data.current?.description,        // 天气描述
          temperature: data.current?.temperature,     // 温度
          // 如果需要体感温度
          // feelsLike: data.current?.feelsLike,
          
          // 如果新的数据结构中没有风向和风力数据，可以设置默认值
          //winddirection: "暂无数据",
          //windpower: "暂无数据",
          
          // 如果需要添加空气质量信息
          //airQuality: {
          //  category: data.current?.airQuality?.category,
          //  statement: data.current?.airQuality?.statement
          //}
      };

    } else {
      // 获取 Adcode
      const adCode = await getAdcode(mainKey);
      console.log(adCode);
      if (adCode.infocode !== "10000") {
        throw "地区查询失败";
      }
      weatherData.adCode = {
        city: adCode.city,
        adcode: adCode.adcode,
      };
      // 获取天气信息
      const result = await getWeather(mainKey, weatherData.adCode.adcode);
      weatherData.weather = {
        weather: result.lives[0].weather,
        temperature: result.lives[0].temperature,
        winddirection: result.lives[0].winddirection,
        windpower: result.lives[0].windpower,
      };
    }
  } catch (error) {
    console.error("天气信息获取失败:" + error);
    onError("天气信息获取失败");
  }
};

// 报错信息
const onError = (message) => {
  ElMessage({
    message,
    icon: h(Error, {
      theme: "filled",
      fill: "#efefef",
    }),
  });
  console.error(message);
};

onMounted(() => {
  // 调用获取天气
  getWeatherData();
});
</script>
