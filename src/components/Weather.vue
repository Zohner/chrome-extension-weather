<template>
  <section>
    <div class="weather" v-if="!loading">
      <div class="weather-hd">
        <p class="city">{{currently.city}}</p>
        <div class="overview">
          <div class="icon">
            <img :src="currently.icon" alt="icon">
          </div>
          <p class="temperature">{{currently.temperature}}</p>
          <ul class="more-list">
            <li class="item">湿度 {{currently.more.dewPoint}}</li>
            <li class="item">降水概率 {{currently.more.precipProbability}}</li>
            <li class="item">紫外线指数 {{currently.more.uvIndex}}</li>
          </ul>
        </div>
        <div class="summary">{{summary}}</div>
      </div>
      <div class="weather-bd">
        <ul class="list">
          <li class="item" v-for="(item,index) in weahter" :key="index" @mouseover="getInfo(index)">
            <p class="week">{{getDay(item.time)}}</p>
            <p class="icon">
              <img :src="iconmap[(item.icon).split('-').join('')]" alt="icon">
            </p>
            <p class="rainfall">{{parseInt(item.precipProbability * 100) + "%"}}</p>
            <p class="lowhigh">{{Math.ceil(item.temperatureMin)}} ˚ / {{Math.ceil(item.temperatureMax)}} ˚</p>
          </li>
        </ul>
      </div>
      <div class="weather-ft">
        Powered by XMit
      </div>
    </div>
    <div class="loading" v-if="loading">
      <Loading></Loading>
    </div>
  </section>
</template>

<script>
import { IconMap } from "../../static/data/icon_map";
import Loading from "@/components/Loading";
export default {
  name: "weather",
  data() {
    return {
      loading: true,
      currently: {
        city: "",
        icon: "",
        temperature: "",
        more: {
          dewPoint: "",
          precipProbability: "",
          uvIndex: ""
        }
      },
      summary: "",
      iconmap: IconMap,
      weahter: []
    };
  },
  methods: {
    getDay(time) {
      let arr = ["周日", "周一", "周二", "周三", "周四", "周五", "周六"];
      return arr[
        new Date(
          new Date(parseInt(time) * 1000).toLocaleString().split(" ")[0]
        ).getDay()
      ];
    },
    getInfo(i) {
      this.summary = this.weahter[i].summary;
    },
    getLocation() {
      let url = "https://ipapi.co/json/";
      this.http
        .get(url)
        .then(it => {
          if (it.status === 200) {
            this.encodeLocation(it.data.longitude, it.data.latitude);
          } else {
            console.log("获取地址失败");
          }
        })
        .catch(e => {
          console.log("getLocation获取接口失败");
        });
    },
    encodeLocation(longitude, latitude) {
      let url = "https://restapi.amap.com/v3/geocode/regeo";
      this.http
        .get(url, {
          params: {
            location: longitude + "," + latitude,
            key: "6bcab73e18c29692d1678e027e7be25a"
          }
        })
        .then(it => {
          if (it.status === 200) {
            this.currently.city =
              it.data.regeocode.addressComponent.province +
              it.data.regeocode.addressComponent.city;
            this.getWeather(latitude, longitude);
          } else {
            console.log("获取地址失败");
          }
        })
        .catch(e => {
          console.log("encodeLocation获取接口失败");
        });
    },
    getWeather(latitude, longitude) {
      let url = `https://api.darksky.net/forecast/b534fc093637c2e5fccdbe93f777fcda/${latitude},${longitude}`;
      this.http
        .get(url, {
          params: {
            units: "si",
            lang: "zh"
          }
        })
        .then(it => {
          this.loading = false;
          if (it.status === 200) {
            this.currently.icon =
              IconMap[it.data.currently.icon.split("-").join("")];
            this.currently.temperature =
              Math.ceil(it.data.currently.temperature) + "˚";
            this.currently.more.dewPoint =
              parseInt(it.data.currently.dewPoint) + "%";
            this.currently.more.precipProbability =
              parseInt(it.data.currently.precipProbability * 100) + "%";
            this.currently.more.uvIndex = it.data.currently.uvIndex;
            this.weahter = it.data.daily.data.slice(1, 6);
            this.summary = it.data.daily.data[1].summary;
          } else {
            console.log("获取天气失败");
          }
        })
        .catch(e => {
          console.log("获取接口失败");
        });
    }
  },
  created() {
    this.getLocation();
  },
  components: {
    Loading
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
.loading {
  width: 600px;
  height: 355px;
}
.weather {
  background: linear-gradient(#2869e9, #79bfff);
  color: #fff;
  border-radius: 5px;
  width: 600px;
  p {
    margin: 0;
  }
  .weather-hd {
    padding: 15px;
    .overview {
      display: flex;
      flex-direction: row;
      flex-wrap: nowrap;
      justify-content: flex-start;
      align-items: center;
      align-content: stretch;
      margin: 10px 0 30px;
      .temperature {
        font-size: 50px;
        margin-left: 15px;
        font-weight: 400;
      }
      .more-list {
        margin-left: 10px;
        li.item {
          margin-bottom: 6px;
        }
      }
    }
  }
  .weather-bd {
    text-align: center;
    border-top: 1px solid #73a9f6;
    padding: 15px;
    .list {
      display: flex;
      flex-direction: row;
      flex-wrap: nowrap;
      justify-content: flex-start;
      align-items: stretch;
      align-content: stretch;
      li.item {
        width: 20%;
        border-left: 1px solid hsla(0, 0%, 100%, 0.2);
        margin-left: -1px;
        .week,
        .rainfall,
        .lowhigh {
          margin-bottom: 10px;
        }
        .icon {
          width: 20px;
          height: 20px;
          display: inline-block;
          margin: 0 5px;
          img {
            width: 24px;
            height: 24px;
          }
        }
        &:first-child {
          border: 0;
        }
      }
    }
  }
  .weather-ft {
    text-align: center;
    padding: 0 0 10px;
    font-size: 12px;
    color: hsla(0, 0%, 100%, 0.6);
  }
}
</style>
