<template>
  <div id="app">
    <div class="row no-gutters">
      <!-- 選擇所在區域 -->
      <div class="toolbox col-sm-3 p-2 bg-white">
        <div class="form-group d-flex">
          <label for="cityName" class="col-form-label mr-2 text-right">縣市</label>
          <div class="flex-fill">
            <select id="cityName" class="form-control" v-model="select.city">
              <option value="">請選擇縣市</option>
              <option
                v-for="(city, index) in cityName" 
                :key="index" 
                :value="city.CityName">
                {{city.CityName}}
              </option>
            </select>
          </div>
        </div>
        <div class="form-group d-flex">
          <label for="area" class="col-form-label mr-2 text-right">地區</label>
          <div class="flex-fill">
            <select id="cityName" class="form-control" v-model="select.area">
              <option value="">請選擇地區</option>
              <option
                v-for="(area,index) in cityName.find(city => city.CityName === select.city).AreaList" :key="index"
                :value="area.AreaName"
              >
              {{area.AreaName}}
              </option>
            </select>
          </div>
        </div>
      </div>
      <!-- 顯示藥局位置 -->
      <div class="col-sm-9">
        <div id="map" style="height: 100vh;"></div>
      </div>
    </div>
  </div>
</template>

<script>
import cityName from '@/assets/cityName.json';
import L from 'leaflet';
let openStreetMap = {}
export default {
  name: 'App',
  data () {
    return {
      maskAPI: 'https://raw.githubusercontent.com/kiang/pharmacies/master/json/points.json',
      cityName,
      select: {
        city: '臺北市',
        area: '中正區'
      },
      data:[]
    }
  },
  mounted () {
    this.createOpenStreetMap();
    this.getMaskData();
  },
  computed: {
    pharmacies() {
      return this.data.filter((phamacy) => {
        if (!this.select.area) {
          return phamacy.properties.county === this.select.city
        } else {
          return phamacy.properties.town === this.select.area
        }
      })
    }
  },
  watch: {
    pharmacies () {
      this.updateMap()
    }
  },
  methods: {
    async getMaskData () {
      const { data:{ features } } = await this.$axios.get(this.maskAPI)
      this.data = features
    },
    createOpenStreetMap () {
      openStreetMap = L.map('map',{
        center: [25.042474, 121.51372],
        zoom: 10
      })
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
        maxZoom: 20,
      }).addTo(openStreetMap);
    },
    updateMap() {
      this.clearMapMarkers()
      this.pharmacies.forEach(pharmacy => {
        L.marker([
          pharmacy.geometry.coordinates[1],
          pharmacy.geometry.coordinates[0],
        ]).addTo(openStreetMap).bindPopup(this.createPharmacyBox(pharmacy))
      })
    },
    clearMapMarkers () {
      openStreetMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          openStreetMap.removeLayer(layer)
        }
      })
    },
    createPharmacyBox (pharmacy) {
        const boxHtml =`
          <p><strong style="font-size: 20px;">${pharmacy.properties.name}</strong></p>
            <strong style="font-size: 16px; color: #d45345;">
                口罩剩餘：成人 - ${pharmacy.properties.mask_adult ? `${pharmacy.properties.mask_adult} 個` : '未取得資料'} / 兒童 - ${pharmacy.properties.mask_child ? `${pharmacy.properties.mask_child} 個` : '未取得資料'}
            </strong>
            <br>
            地址: ${pharmacy.properties.address}<br>
            電話: ${pharmacy.properties.phone}<br>
            <small>最後更新時間: ${pharmacy.properties.updated}</small>`
            return boxHtml
    }
  }
}
</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';
#map {
  position: relative;
  height: 100vh;
}
</style>
