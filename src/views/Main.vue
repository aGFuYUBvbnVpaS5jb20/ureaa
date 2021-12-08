<template>
  <v-app>
    <Bar />
    <Nav />
    <v-main>
      <vue-daum-map
        :appKey="appKey"
        :center.sync="center"
        :level.sync="level"
        :mapTypeId="mapTypeId"
        :libraries="libraries"
        @load="load"
        @center_changed="request()"
        style="width: 100vw; height: 100%"
      />
      <v-toolbar floating absolute class="ma-3">
        <v-text-field
          hide-details
          single-line
          clearable
          color="blue accent-3"
          v-model="addr"
          placeholder="장소 검색"
          prepend-icon="mdi-magnify"
          @keyup.native.enter="search()"
        ></v-text-field>
        <v-tooltip bottom>
          <template v-slot:activator="{ on }">
            <v-btn icon @click="location()" v-on="on">
              <v-icon> mdi-crosshairs-gps </v-icon>
            </v-btn>
          </template>
          <span>내 위치 찾기</span>
        </v-tooltip>
        <v-tooltip bottom>
          <template v-slot:activator="{ on }">
            <v-btn icon @click="refresh()" v-on="on">
              <v-icon> mdi-autorenew </v-icon>
            </v-btn>
          </template>
          <span>새로고침</span>
        </v-tooltip>
        <v-tooltip bottom>
          <template v-slot:activator="{ on }">
            <v-switch
              hide-details
              inset
              v-model="empty"
              color="orange"
              v-on="on"
              class="ml-3"
              @change="refresh()"
            ></v-switch>
          </template>
          <span>소진된 장소 숨기기</span>
        </v-tooltip>
      </v-toolbar>
    </v-main>
  </v-app>
</template>

<script
  type="text/javascript"
  src="//dapi.kakao.com/v2/maps/sdk.js?appkey=bf5196e1c2e4826836d8d5a8e6d25a3f&libraries=services"
></script>

<script>
import Bar from "@/components/Bar";
import Nav from "@/components/Nav";
import VueDaumMap from "vue-daum-map";

export default {
  name: "Main",

  components: {
    Bar,
    Nav,
    VueDaumMap,
  },

  data: () => ({
    appKey: "bf5196e1c2e4826836d8d5a8e6d25a3f",
    center: { lat: 37.507587841577454, lng: 127.06057692858644 },
    lat: 0,
    lng: 0,
    level: 9,
    mapTypeId: VueDaumMap.MapTypeId.NORMAL,
    libraries: ["services"],
    addr: null,
    buttonOverlay: [],
    cardOverlay: [],
    empty: false,
  }),

  methods: {
    load(map) {
      this.map = map;
      this.request();
      this.location();
    },
    search() {
      var ps = new kakao.maps.services.Places();
      ps.keywordSearch(this.addr, placesSearchCB);

      function placesSearchCB(data, status) {
        if (status === kakao.maps.services.Status.OK) {
          setCenter(data[0].y, data[0].x);
        }
      }

      let setCenter = (lat, lon) => {
        this.map.setLevel(9);
        this.map.setCenter(new kakao.maps.LatLng(lat, lon));
        this.request();
      };
    },
    location() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(function (pos) {
          setCenter(pos.coords.latitude, pos.coords.longitude);
        });
      }

      let setCenter = (lat, lng) => {
        this.map.setLevel(9);
        this.map.setCenter(new kakao.maps.LatLng(lat, lng));
        this.request();
      };
    },
    refresh() {
      this.lat = 0;
      this.lng = 0;
      this.remove();
      this.request();
    },
    request() {
      if (
        this.lat - 0.03 > this.center.lat ||
        this.lat + 0.03 < this.center.lat ||
        this.lng - 0.03 > this.center.lng ||
        this.lng + 0.03 < this.center.lng
      ) {
        this.lat = this.center.lat;
        this.lng = this.center.lng;
        axios
          .get(
            `https://api.odcloud.kr/api/uws/v1/inventory?page=1&perPage=1000&serviceKey=Tv4JRCwAhanGg%2FwqM7A%2FDjD8ar5owZG2c6rV%2FiIwZiXKe%2BRBvf%2FGqwxGPI3CE8FuCGIkCAVFfcllGi7qMObEbA%3D%3D`
          )
          .then((res) => {
            res.data.data.forEach((e) => {
              if (this.empty ? e.inventory !== "0" : true) {
                let color;
                if (e.inventory === "0") {
                  color = "dark";
                } else if (Number(e.inventory) < 300) {
                  color = "danger";
                } else if (Number(e.inventory) < 1000) {
                  color = "warning";
                } else {
                  color = "success";
                }

                const button = `
                <button id="btn${e.code}" class="btn btn-sm btn-${color}" onclick="document.getElementById('card${e.code}').style.display = 'block'; document.getElementById('btn${e.code}').style.display = 'none';">${e.inventory}L</button>
              `;

                const card = `
                <div id="card${
                  e.code
                }" class="card" style="margin-left: -50%; width: 100%; min-width: 260px; display: none;">
                  <div class="card-body">
                    <button class="close" onclick="document.getElementById('card${
                      e.code
                    }').style.display = 'none'; document.getElementById('btn${
                  e.code
                }').style.display = 'block'">
                      <span aria-hidden="true">&times;</span>
                    </button>
                    <small class="text-dark">${
                      e.regDt ? e.regDt + " 기준" : "기준 정보 없음"
                    }</small>
                    <h5 class="card-title text-dark"><button class="btn btn-sm btn-${color}">${
                  e.inventory
                }L</button> ${e.name}</h5>
                    <h6 class="card-text text-dark" style="white-space: normal;">${
                      e.addr
                    }</h6>
                    <h6 class="card-text text-dark">${
                      e.price ? "리터당 " + e.price + "원" : "가격 정보 없음"
                    }</h6>
                    <h6 class="card-text text-dark">${
                      e.openTime ? e.openTime : "시간 정보 없음"
                    }</h6>
                    <h6 class="card-text text-dark">${
                      e.tel ? e.tel : "연락처 정보 없음"
                    }</h6>
                    <a href="https://map.kakao.com/link/to/${e.name},${e.lat},${
                  e.lng
                }" target="_blank" style="text-decoration: none;"><button class="btn btn-outline-primary btn-block">길찾기</button></a>
                  </div>
                </div>
              `;

                let pos = new kakao.maps.LatLng(e.lat, e.lng);

                this.buttonOverlay.push(
                  new kakao.maps.CustomOverlay({
                    map: this.map,
                    position: pos,
                    content: button,
                  })
                );

                this.cardOverlay.push(
                  new kakao.maps.CustomOverlay({
                    map: this.map,
                    position: pos,
                    content: card,
                    zIndex: 1,
                  })
                );
              }
            });
          });
      }
    },
    remove() {
      this.buttonOverlay.forEach((e) => {
        e.setMap(null);
      });
      this.cardOverlay.forEach((e) => {
        e.setMap(null);
      });
      this.buttonOverlay = [];
      this.cardOverlay = [];
    },
  },
};
</script>
