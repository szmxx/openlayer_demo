<template>
  <div class="container">
    <div class="button-container">
      <button @click="locateToLondun">lundon</button>
    </div>
    <div id="map"></div>
  </div>
</template>

<script>
import { Map, View } from "ol";
import Feature from "ol/Feature";
import { LineString, Point, Polygon } from "ol/geom";
import { Tile as TileLayer, Vector as VectorLayer } from "ol/layer";
import { unByKey } from "ol/Observable";
import {
  OSM,
  TileDebug,
  TileArcGISRest,
  Vector as VectorSource
} from "ol/source";
import {
  Fill,
  /*Icon,*/ Circle as CircleStyle,
  Stroke,
  Style,
  Text
} from "ol/style";
import {
  Draw,
  Modify,
  Snap,
  Select,
  defaults as defaultInteractions,
  DragRotateAndZoom
} from "ol/interaction";
import { defaults as defaultControls, ScaleLine, ZoomSlider } from "ol/control";
// 将经纬度转为投影坐标系
// import { fromLonLat } from "ol/proj";
import { easeIn, easeOut } from "ol/easing";
import { getVectorContext } from "ol/render";
export default {
  name: "OLMap",
  data() {
    return {
      map: null,
      view: null
    };
  },
  mounted() {
    this.initMap();
  },
  methods: {
    initMap() {
      let arcgisLayerUrl =
        "https://sampleserver1.arcgisonline.com/ArcGIS/rest/services/" +
        "Specialty/ESRI_StateCityHighway_USA/MapServer";
      let pointFeature = new Feature(new Point([0, 0])),
        polygonFeature = new Feature(
          new Polygon([
            [
              [-30, 40],
              [-32, 30],
              [-25, 50],
              [-20, 60],
              [-30, 40]
            ]
          ])
        ),
        lineFeature = new Feature(
          new LineString([
            [-20, 30],
            [-30, 40]
          ])
        );
      var vectorSource = new VectorSource();
      let tileLayer = new TileLayer({
        source: new OSM({
          wrapX: false
        })
      });
      this.view = new View({
        center: [0, 0],
        zoom: 3,
        // 转换成4326坐标系
        projection: "EPSG:4326"
      });
      this.map = new Map({
        interactions: defaultInteractions().extend([new DragRotateAndZoom()]),
        controls: defaultControls().extend([
          new ScaleLine({
            units: "degrees"
          }),
          new ZoomSlider()
        ]),
        target: "map",
        layers: [
          tileLayer,
          new TileLayer({
            source: new TileDebug()
          }),
          new TileLayer({
            extent: [
              -183.780014728684,
              16.3007091216188,
              -61.4068546696842,
              74.0303080309689
            ],
            source: new TileArcGISRest({
              url: arcgisLayerUrl
            })
          }),
          new VectorLayer({
            source: vectorSource,
            style: new Style({
              image: new CircleStyle({
                radius: 10,
                stroke: new Stroke({
                  color: "#fff"
                }),
                fill: new Fill({
                  color: "#3399CC"
                })
              }),
              stroke: new Stroke({
                width: 3,
                color: [255, 0, 0, 1]
              }),
              fill: new Fill({
                color: [0, 0, 255, 0.6]
              }),
              text: new Text({
                text: "simple",
                fill: new Fill({
                  color: "#fff"
                })
              })
            })
          })
        ],
        view: this.view
      });
      // this.initDraw(vectorSource);
      this.addLayer([pointFeature, lineFeature, polygonFeature]);
      this.modifySelect([pointFeature, lineFeature, polygonFeature]);
      this.flash(pointFeature, tileLayer);
    },
    initDraw(layer, isModify = true, freehand = false) {
      // type: Point,Circle,Polygon
      this.draw = new Draw({
        source: layer,
        freehand: freehand,
        type: "LineString"
      });
      this.map.addInteraction(this.draw);
      if (isModify) {
        let modify = new Modify({
          source: layer
        });
        this.map.addInteraction(modify);
        let snap = new Snap({ source: layer });
        this.map.addInteraction(snap);
      }
      this.draw.on("drawend", function(event) {
        let feature = event.feature;
        if (feature) {
          console.log("你绘制了几何" + feature);
        }
      });
    },
    addLayer(feature) {
      let features = [];
      if (Array.isArray(feature)) {
        features.push(...feature);
      } else {
        features.push(feature);
      }
      let vectorSource = new VectorSource({
        features: features
      });
      this.map.addLayer(
        new VectorLayer({
          source: vectorSource
        })
      );
      this.map.getView().fit(vectorSource.getExtent());
    },
    modifySelect() {
      let select = new Select();
      console.log(select.getFeatures());
      let modify = new Modify({
        features: select.getFeatures()
      });
      this.map.addInteraction(select);
      this.map.addInteraction(modify);
    },
    locateToLondun() {
      let london = [-0.12755, 51.507222];
      console.log(london);
      this.view.animate({
        center: london,
        duration: 2000,
        easing: easeIn
      });
    },
    flash(feature, tileLayer) {
      let duration = 3000;
      var start = new Date().getTime();
      var listenerKey = tileLayer.on("postrender", animate);
      function animate(event) {
        var vectorContext = getVectorContext(event);
        var frameState = event.frameState;
        var flashGeom = feature.getGeometry().clone();
        var elapsed = frameState.time - start;
        var elapsedRatio = elapsed / duration;
        // radius will be 5 at start and 30 at end.
        var radius = easeOut(elapsedRatio) * 25 + 5;
        var opacity = easeOut(1 - elapsedRatio);

        var style = new Style({
          image: new CircleStyle({
            radius: radius,
            stroke: new Stroke({
              color: "rgba(255, 0, 0, " + opacity + ")",
              width: 0.25 + opacity
            })
          })
        });

        vectorContext.setStyle(style);
        vectorContext.drawGeometry(flashGeom);
        if (elapsed > duration) {
          unByKey(listenerKey);
          return;
        }
        this.map.render();
      }
    }
  }
};
</script>
<style scoped>
.container {
  position: relative;
  width: 100%;
  height: 100%;
}
#map {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
.button-container {
  position: absolute;
  display: flex;
  justify-content: center;
  align-content: center;
  z-index: 99;
}
</style>
