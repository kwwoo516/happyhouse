<template>
  <div>
     <md-content class="md-elevation-6">
    <div class="map_wrap">
      <div
        id="map"
        style="width: 100%; height: 100%; position: relative; overflow: hidden"
      ></div>
      <ul id="category">
        <li
          id="BK9"
          data-order="0"
          ref="cg0"
          v-on:click="onClickCategory('cg0')"
        >
          <span class="category_bg bank"></span>
          은행
        </li>
        <li
          id="MT1"
          data-order="1"
          ref="cg1"
          v-on:click="onClickCategory('cg1')"
        >
          <span class="category_bg mart"></span>
          마트
        </li>
        <li
          id="PM9"
          data-order="2"
          ref="cg2"
          v-on:click="onClickCategory('cg2')"
        >
          <span class="category_bg pharmacy"></span>
          약국
        </li>
        <li
          id="OL7"
          data-order="3"
          ref="cg3"
          v-on:click="onClickCategory('cg3')"
        >
          <span class="category_bg oil"></span>
          주유소
        </li>
        <li
          id="CE7"
          data-order="4"
          ref="cg4"
          v-on:click="onClickCategory('cg4')"
        >
          <span class="category_bg cafe"></span>
          카페
        </li>
        <li
          id="CS2"
          data-order="5"
          ref="cg5"
          v-on:click="onClickCategory('cg5')"
        >
          <span class="category_bg store"></span>
          편의점
        </li>
      </ul>
      <div class="button-group">
        <button @click="markCovidCenter">
          <img
            src="../assets/img/covidCenter.png"
            height="30"
            width="30"
          />코로나19
        </button>
        <button @click="hideCovidCenter">
          <img src="../assets/img/eraser.png" height="45" width="45" />지우기
        </button>
      </div>
    </div>
    </md-content>
  </div>
</template>
<script>
import { mapState, mapActions, mapMutations } from "vuex";

const houseStore = "houseStore";
const covidCenterStore = "covidCenterStore";

export default {
  name: "KakaoMap",
  data() {
    return {
      overlay: null,
      placeOverlay: null,
      map: null,
      ps: null,
      contentNode: null,
      markers: [],
      covidmarkers: [],
      housemarkers: [],
      currCategory: "",
      infowindow: null,
    };
  },
  computed: {
    ...mapState(covidCenterStore, ["covidcenters"]),
    ...mapState(houseStore, ["houses"]),
  },
  mounted() {
    if (window.kakao && window.kakao.maps) {
      this.initMap();
    } else {
      const script = document.createElement("script");
      /* global kakao */
      script.onload = () => kakao.maps.load(this.initMap);
      script.src =
        "//dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=5910f388d8ea76969320c90e392024b2&libraries=services";
      document.head.appendChild(script);
    }
  },
  watch: {
    houses() {
      this.markHouse();
    },
  },
  methods: {
    ...mapActions(covidCenterStore, ["getCovidCenterList"]),
    markHouse() {
      if (this.housemarkers.length > 0) {
        for (var i = 0; i < this.housemarkers.length; i++) {
          this.housemarkers[i].setMap(null);
        }
        this.housemarkers = [];
      }
      for (let i = 0; i < this.houses.length; i++) {
        var addr =
          "서울특별시 " + this.houses[i].법정동 + " " + this.houses[i].지번;
        var geocoder = new kakao.maps.services.Geocoder();
        geocoder.addressSearch(addr, (result, status) => {
          // 정상적으로 검색이 완료됐으면
          if (status === kakao.maps.services.Status.OK) {
            var coords = new kakao.maps.LatLng(result[0].y, result[0].x);
            // 결과값으로 받은 위치를 마커로 표시합니다
            var marker = new kakao.maps.Marker({
              map: this.map,
              title: this.houses[i].아파트,
              position: coords,
            });

            // 인포윈도우로 장소에 대한 설명을 표시합니다
            var infowindow = new kakao.maps.InfoWindow({
              content:
                '<div style="width:150px;text-align:center;padding:6px 0;">우리회사</div>',
            });

            // 지도의 중심을 결과값으로 받은 위치로 이동시킵니다
            //this.map.setCenter(coords);
            marker.setMap(this.map);
            this.housemarkers.push(marker);

            if (i == this.houses.length - 1) {
              this.map.setCenter(coords);
            }
          }
        });
      }
    },
    markCovidCenter() {
      if (this.covidmarkers.length <= 0) {
        this.getCovidCenterList();
        const imageSrc =
          "https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/markerStar.png";
        const imageSize = new kakao.maps.Size(24, 35);

        for (let i = 0; i < this.covidcenters.length; i++) {
          const markerImage = new kakao.maps.MarkerImage(imageSrc, imageSize);

          var placePosition = new kakao.maps.LatLng(
            this.covidcenters[i].lat,
            this.covidcenters[i].lng
          );
          // 마커 생성
          let marker = new kakao.maps.Marker({
            map: this.map,
            title: this.covidcenters[i].centerName,
            position: placePosition,
            image: markerImage, // 마커이미지 설정
          });
          //console.dir(marker.position);
          marker.setMap(this.map);
          this.covidmarkers.push(marker);
        }
      } else {
        this.setMarkers(this.map);
      }
    },
    setMarkers(map) {
      for (let i = 0; i < this.covidmarkers.length; i++) {
        this.covidmarkers[i].setMap(map);
      }
    },
    hideCovidCenter() {
      this.setMarkers(null);
    },
    initMap() {
      this.placeOverlay = new kakao.maps.CustomOverlay({ zIndex: 1 });
      this.contentNode = document.createElement("div");
      const container = document.getElementById("map");
      const options = {
        center: new kakao.maps.LatLng(37.566826, 126.9786567),
        level: 5,
      };
      this.map = new kakao.maps.Map(container, options);
      this.ps = new kakao.maps.services.Places(this.map);

      kakao.maps.event.addListener(this.map, "idle", this.searchPlaces);
      this.contentNode.className = "placeinfo_wrap";

      this.addEventHandle(
        this.contentNode,
        "touchstart",
        kakao.maps.event.preventMap
      );
      this.addEventHandle(
        this.contentNode,
        "mousedown",
        kakao.maps.event.preventMap
      );

      this.placeOverlay.setContent(this.contentNode);
      //this.addCategoryClickEvent();

      //아파트마커
      // this.overlay = new kakao.maps.CustomOverlay({
      //   content: content,
      //   map: map,
      //   position: marker.getPosition(),
      // });
    },
    addEventHandle(target, type, callback) {
      if (target.addEventListener) {
        target.addEventListener(type, callback);
      } else {
        target.attachEvent("on" + type, callback);
      }
    },
    searchPlaces() {
      if (!this.currCategory) {
        return;
      }
      // 커스텀 오버레이를 숨깁니다
      this.placeOverlay.setMap(null);
      // 지도에 표시되고 있는 마커를 제거합니다
      this.removeMarker();
      this.ps.categorySearch(this.currCategory, this.placesSearchCB, {
        useMapBounds: true,
      });
    },
    placesSearchCB(data, status, pagination) {
      if (status === kakao.maps.services.Status.OK) {
        // 정상적으로 검색이 완료됐으면 지도에 마커를 표출합니다
        this.displayPlaces(data);
      } else if (status === kakao.maps.services.Status.ZERO_RESULT) {
        // 검색결과가 없는경우 해야할 처리가 있다면 이곳에 작성해 주세요
      } else if (status === kakao.maps.services.Status.ERROR) {
        // 에러로 인해 검색결과가 나오지 않은 경우 해야할 처리가 있다면 이곳에 작성해 주세요
      }
    },
    // 클릭한 마커에 대한 장소 상세정보를 커스텀 오버레이로 표시하는 함수입니다
    displayPlaceInfo(place) {
      var content =
        '<div class="placeinfo">' +
        '   <a class="title" href="' +
        place.place_url +
        '" target="_blank" title="' +
        place.place_name +
        '">' +
        place.place_name +
        "</a>";

      if (place.road_address_name) {
        content +=
          '    <span title="' +
          place.road_address_name +
          '">' +
          place.road_address_name +
          "</span>" +
          '  <span class="jibun" title="' +
          place.address_name +
          '">(지번 : ' +
          place.address_name +
          ")</span>";
      } else {
        content +=
          '    <span title="' +
          place.address_name +
          '">' +
          place.address_name +
          "</span>";
      }

      content +=
        '    <span class="tel">' +
        place.phone +
        "</span>" +
        "</div>" +
        '<div class="after"></div>';

      this.contentNode.innerHTML = content;
      this.placeOverlay.setPosition(new kakao.maps.LatLng(place.y, place.x));
      this.placeOverlay.setMap(this.map);
    },
    // 지도에 마커를 표출하는 함수입니다
    displayPlaces(places) {
      // 몇번째 카테고리가 선택되어 있는지 얻어옵니다
      // 이 순서는 스프라이트 이미지에서의 위치를 계산하는데 사용됩니다
      var order = document
        .getElementById(this.currCategory)
        .getAttribute("data-order");
      for (var i = 0; i < places.length; i++) {
        // 마커를 생성하고 지도에 표시합니다
        var marker = this.addMarker(
          new kakao.maps.LatLng(places[i].y, places[i].x),
          order
        );

        // 마커와 검색결과 항목을 클릭 했을 때
        // 장소정보를 표출하도록 클릭 이벤트를 등록합니다

        //kakao.maps.event.addListener(marker, "click", this.displayPlaceInfo);
        function displayPlaceInfo(param, place) {
          var content =
            '<div class="placeinfo">' +
            '   <a class="title" href="' +
            place.place_url +
            '" target="_blank" title="' +
            place.place_name +
            '">' +
            place.place_name +
            "</a>";

          if (place.road_address_name) {
            content +=
              '    <span title="' +
              place.road_address_name +
              '">' +
              place.road_address_name +
              "</span>" +
              '  <span class="jibun" title="' +
              place.address_name +
              '">(지번 : ' +
              place.address_name +
              ")</span>";
          } else {
            content +=
              '    <span title="' +
              place.address_name +
              '">' +
              place.address_name +
              "</span>";
          }

          content +=
            '    <span class="tel">' +
            place.phone +
            "</span>" +
            "</div>" +
            '<div class="after"></div>';

          param.contentNode.innerHTML = content;
          param.placeOverlay.setPosition(
            new kakao.maps.LatLng(place.y, place.x)
          );
          param.placeOverlay.setMap(param.map);
        }
        (function (param, marker, place) {
          kakao.maps.event.addListener(marker, "click", function () {
            displayPlaceInfo(param, place);
          });
        })(this, marker, places[i]);
      }
    },
    // 마커를 생성하고 지도 위에 마커를 표시하는 함수입니다
    addMarker(position, order) {
      var imageSrc =
          "https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/places_category.png", // 마커 이미지 url, 스프라이트 이미지를 씁니다
        imageSize = new kakao.maps.Size(27, 28), // 마커 이미지의 크기
        imgOptions = {
          spriteSize: new kakao.maps.Size(72, 208), // 스프라이트 이미지의 크기
          spriteOrigin: new kakao.maps.Point(46, order * 36), // 스프라이트 이미지 중 사용할 영역의 좌상단 좌표
          offset: new kakao.maps.Point(11, 28), // 마커 좌표에 일치시킬 이미지 내에서의 좌표
        },
        markerImage = new kakao.maps.MarkerImage(
          imageSrc,
          imageSize,
          imgOptions
        ),
        marker = new kakao.maps.Marker({
          position: position, // 마커의 위치
          image: markerImage,
        });

      marker.setMap(this.map); // 지도 위에 마커를 표출합니다
      this.markers.push(marker); // 배열에 생성된 마커를 추가합니다

      return marker;
    },
    // 지도 위에 표시되고 있는 마커를 모두 제거합니다
    removeMarker() {
      for (var i = 0; i < this.markers.length; i++) {
        this.markers[i].setMap(null);
      }
      this.markers = [];
    },
    // 각 카테고리에 클릭 이벤트를 등록합니다
    addCategoryClickEvent() {
      var category = document.getElementById("category"),
        children = category.children;

      for (var i = 0; i < children.length; i++) {
        children[i].onclick = this.onClickCategory;
      }
    },
    // 카테고리를 클릭했을 때 호출되는 함수입니다
    onClickCategory(param) {
      var id = this.$refs[param].id,
        className = this.$refs[param].className;

      this.placeOverlay.setMap(null);
      if (className === "on") {
        this.currCategory = "";
        this.changeCategoryClass();
        this.removeMarker();
      } else {
        this.currCategory = id;
        this.changeCategoryClass(this.$refs[param]);
        this.searchPlaces();
      }
    },
    // 클릭된 카테고리에만 클릭된 스타일을 적용하는 함수입니다
    changeCategoryClass(el) {
      var category = document.getElementById("category"),
        children = category.children,
        i;

      for (i = 0; i < children.length; i++) {
        children[i].className = "";
      }
      if (el) {
        el.className = "on";
      }
    },
    closeOverlay() {
      this.overlay.setMap(null);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
#map {
  width: 690px;
  height: 600px;
}

.button-group {
  margin: 10px 0px;
}

button {
  margin: 0 3px;
}
.map_wrap,
.map_wrap * {
  margin: 0;
  padding: 0;
  font-family: "Malgun Gothic", dotum, "돋움", sans-serif;
  font-size: 12px;
}
.map_wrap {
  position: relative;
  width: 100%;
  height: 600px;
}
#category {
  position: absolute;
  top: 10px;
  left: 10px;
  border-radius: 5px;
  border: 1px solid #909090;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.4);
  background: #fff;
  overflow: hidden;
  z-index: 2;
}
#category li {
  float: left;
  list-style: none;
  width: 50px;
  border-right: 1px solid #acacac;
  padding: 6px 0;
  text-align: center;
  cursor: pointer;
}
#category li.on {
  background: #eee;
}
#category li:hover {
  background: #ffe6e6;
  border-left: 1px solid #acacac;
  margin-left: -1px;
}
#category li:last-child {
  margin-right: 0;
  border-right: 0;
}
#category li span {
  display: block;
  margin: 0 auto 3px;
  width: 27px;
  height: 28px;
}
#category li .category_bg {
  background: url(https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/places_category.png)
    no-repeat;
}
#category li .bank {
  background-position: -10px 0;
}
#category li .mart {
  background-position: -10px -36px;
}
#category li .pharmacy {
  background-position: -10px -72px;
}
#category li .oil {
  background-position: -10px -108px;
}
#category li .cafe {
  background-position: -10px -144px;
}
#category li .store {
  background-position: -10px -180px;
}
#category li.on .category_bg {
  background-position-x: -46px;
}
.placeinfo_wrap {
  position: absolute;
  bottom: 28px;
  left: -150px;
  width: 300px;
}
.placeinfo {
  position: relative;
  width: 100%;
  border-radius: 6px;
  border: 1px solid #ccc;
  border-bottom: 2px solid #ddd;
  padding-bottom: 10px;
  background: #fff;
}
.placeinfo:nth-of-type(n) {
  border: 0;
  box-shadow: 0px 1px 2px #888;
}
.placeinfo_wrap .after {
  content: "";
  position: relative;
  margin-left: -12px;
  left: 50%;
  width: 22px;
  height: 12px;
  background: url("https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/vertex_white.png");
}
.placeinfo a,
.placeinfo a:hover,
.placeinfo a:active {
  color: #fff;
  text-decoration: none;
}
.placeinfo a,
.placeinfo span {
  display: block;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
.placeinfo span {
  margin: 5px 5px 0 5px;
  cursor: default;
  font-size: 13px;
}
.placeinfo .title {
  font-weight: bold;
  font-size: 14px;
  border-radius: 6px 6px 0 0;
  margin: -1px -1px 0 -1px;
  padding: 10px;
  color: #fff;
  background: #d95050;
  background: #d95050
    url(https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/arrow_white.png)
    no-repeat right 14px center;
}
.placeinfo .tel {
  color: #0f7833;
}
.placeinfo .jibun {
  color: #999;
  font-size: 11px;
  margin-top: 0;
}
</style>
