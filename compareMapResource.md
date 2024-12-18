在cesium中可以引入不同地图资源，这里介绍OpenStreetMap、天地图、bingmap、ArcGI的方式。
cesium版本为V1.108
# 新建cesium viewer
`
import * as Cesium from 'cesium'

window.Cesium = Cesium
const viewer = new Cesium.Viewer('container', {
    shouldAnimate: true, // 开启动画
    selectionIndicator: false, // 显示选择指示器
    baseLayerPicker: false, // 显示图层选择器
    fullscreenButton: false, // 显示全屏按钮
    geocoder: false, // 显示地址搜索
    homeButton: false, // 显示主页按钮
    infoBox: false, // 显示信息框
    sceneModePicker: false, // 显示场景模式选择器
    timeline: false, // 显示时间轴
    navigationHelpButton: false, // 显示导航帮助按钮
    navigationInstructionsInitiallyVisible: false, // 导航帮助按钮是否默认显示
    showRenderLoopErrors: false, // 显示渲染循环错误
    shadows: false, // 显示阴影
    terrain: Cesium.Terrain.fromWorldTerrain(), // 地形数据
    // 关闭无级缩放地图
    smoothResolutionConstraint: false, //平滑分辨率约束
    zoomFactor: 50, // 缩放因子
    zoom: 12, // 缩放级别
})

// Initialize default viewer settings
viewer.scene.globe.depthTestAgainstTerrain = true
viewer.scene.globe.showGroundAtmosphere = false
viewer.scene.globe.enableLighting = false
viewer.scene.debugShowFramesPerSecond = false
viewer.scene.fxaa = false
viewer.scene.postProcessStages.fxaa.enabled = false
// 水雾特效,true会让海洋颜色变浅蓝
viewer.scene.globe.showGroundAtmosphere = false
// 设置最大俯仰角，[-90,0]区间内，默认为-30，单位弧度
viewer.scene.screenSpaceCameraController.constrainedPitch =
  Cesium.Math.toRadians(-20)
viewer.scene.screenSpaceCameraController.autoResetHeadingPitch = false
viewer.scene.screenSpaceCameraController.inertiaZoom = 0.5
viewer.scene.screenSpaceCameraController.minimumZoomDistance = 50
viewer.scene.screenSpaceCameraController.maximumZoomDistance = 20000000
viewer.scene.fog.enabled = false
viewer.scene.sun.show = false
viewer.scene.skyBox.show = false
viewer.scene.skyAtmosphere.show = false

viewer.scene.screenSpaceCameraController.zoomEventTypes = [
  Cesium.CameraEventType.RIGHT_DRAG,
  Cesium.CameraEventType.WHEEL,
  Cesium.CameraEventType.PINCH,
]

viewer.scene.screenSpaceCameraController.tiltEventTypes = [
  Cesium.CameraEventType.MIDDLE_DRAG,
  Cesium.CameraEventType.PINCH,
  {
    eventType: Cesium.CameraEventType.LEFT_DRAG,
    modifier: Cesium.KeyboardEventModifier.CTRL,
  },
  {
    eventType: Cesium.CameraEventType.RIGHT_DRAG,
    modifier: Cesium.KeyboardEventModifier.CTRL,
  },
]

viewer.cesiumWidget.screenSpaceEventHandler.removeInputAction(
  Cesium.ScreenSpaceEventType.LEFT_DOUBLE_CLICK,
)

// 控制不展示动画组件和时间线组件
viewer._cesiumWidget._creditContainer.style.display = 'none' // 隐藏cesium ion
if (viewer.timeline) {
  viewer.timeline.container.style.display = 'none' // 隐藏时间线
}
if (viewer.animation) {
  viewer.animation.container.style.visibility = 'hidden' // 隐藏动画控件
}
`


# 地图引入
## 天地图
`
const tdtUrl = 'https://t{s}.tianditu.gov.cn/'
const tdtToken = '你的天地图token'
const subdomains = ['0', '1', '2', '3', '4', '5', '6', '7']
// 叠加影像服务
const imgMap = new Cesium.UrlTemplateImageryProvider({
  url: tdtUrl + 'DataServer?T=img_w&x={x}&y={y}&l={z}&tk=' + token,
  subdomains: subdomains,
  tilingScheme: new Cesium.WebMercatorTilingScheme(),
  maximumLevel: 18,
})
viewer.imageryLayers.addImageryProvider(imgMap)

// 叠加国界服务
const iboMap = new Cesium.UrlTemplateImageryProvider({
  url: tdtUrl + 'DataServer?T=ibo_w&x={x}&y={y}&l={z}&tk=' + token,
  subdomains: subdomains,
  tilingScheme: new Cesium.WebMercatorTilingScheme(),
  maximumLevel: 10,
})
viewer.imageryLayers.addImageryProvider(iboMap)

// 叠加矢量注记服务

const vector_marker = new Cesium.WebMapTileServiceImageryProvider({
  url:
    tdtUrl +
    'cva_w/wmts?service=wmts&request=GetTile&version=1.0.0&LAYER=cva&tileMatrixSet=w&TileMatrix={TileMatrix}&TileRow={TileRow}&TileCol={TileCol}&style=default.jpg&tk=' +
    token,
  layer: 'tdtAnnoLayer',
  style: 'default',
  subdomains: subdomains,
  format: 'image/jpeg',
  tileMatrixSetID: 'GoogleMapsCompatible',
})
viewer.imageryLayers.addImageryProvider(vector_marker)  
`
## bing Map
由于cesium默认调用的是bing map，即使不添加下述代码，也会渲染bingmap地图
`
const tdtLayer = new Cesium.BingMapsImageryProvider({
  url: 'https://ecn.t{s}.tiles.virtualearth.net/tiles/h{q}.jpeg?n=z&g=11404&mkt=',
  subdomains: ['0', '1', '2', '3'],
  tilingScheme: new Cesium.WebMercatorTilingScheme(),
  customTags: {
    // q: function (imageryProvider, x, y, level) {
    //   const result = tileXYToQuadKey(x, y, level)
    //   console.log(imageryProvider, x, y, level, result)
    //   return result
    // },
  },
})
viewer.imageryLayers.addImageryProvider(tdtLayer)
`

## ArcGIS
`
const imageryLayer = new Cesium.WebMapTileServiceImageryProvider({
  url: 'https://services.arcgisonline.com/arcgis/rest/services/World_Imagery/MapServer/WMTS',
  layer: 'World_Imagery',
  style: 'default',
  format: 'image/jpeg',
  tileMatrixSetID: 'GoogleMapsCompatible',
  maximumLevel: 19,
  credit: new Cesium.Credit('© Esri', 'https://www.esri.com/'),
})
viewer.imageryLayers.addImageryProvider(imageryLayer)

// 添加路网图层
const roadsLayer = await Cesium.ArcGisMapServerImageryProvider.fromUrl(
  'https://services.arcgisonline.com/ArcGIS/rest/services/Reference/World_Transportation/MapServer',
)
viewer.imageryLayers.addImageryProvider(roadsLayer)

// // 添加国界线和地名标注图层
const boundariesAndLabelsLayer = await Cesium.ArcGisMapServerImageryProvider.fromUrl(
  'https://services.arcgisonline.com/ArcGIS/rest/services/Reference/World_Boundaries_and_Places/MapServer',
)
viewer.imageryLayers.addImageryProvider(boundariesAndLabelsLayer)
`

## OpenStreetMap
`
const bingMap = new Cesium.UrlTemplateImageryProvider({
  url: 'https://tile.openstreetmap.bzh/br/{z}/{x}/{y}.png',
  subdomains: ['a', 'b', 'c', 'd'],
})
viewer.imageryLayers.addImageryProvider(bingMap)
`
