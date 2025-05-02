<template>
  <div class="hello">
    <div id="cesiumContainer"></div>
    <div id="showCoordinates" class="labels">

    </div>
    <!-- 公交按钮 -->
    <div class="bus-button" @click="toggleBusSchedule">
      <img src="/images/icon-bus.png" alt="Bus Icon" />
    </div>
    
    <!-- 公交弹窗 -->
    <div class="bus-schedule-overlay" v-if="showBusSchedule">
      <div class="bus-schedule-container" ref="scheduleContainer">
        <div class="close-button" @click="closeBusSchedule">×</div>
        <div class="schedule-image-container" 
          @wheel.prevent="handleWheel"
          @mousedown="handleMouseDown"
          @mousemove="handleMouseMove"
          @mouseup="handleMouseUp"
          @mouseleave="handleMouseUp">
          <img 
            ref="scheduleImage" 
            src="/images/bus-schedule.png" 
            alt="Bus Schedule" 
            :style="{
              transform: `translate(${imagePosition.x}px, ${imagePosition.y}px) scale(${imageScale})`,
              cursor: isDragging ? 'grabbing' : 'grab'
            }"
            @touchstart="handleTouchStart"
            @touchmove="handleTouchMove"
            @touchend="handleTouchEnd"
            draggable="false"
          />
        </div>
      </div>
    </div>

    <table class="labels" id="sceneOption">
      <tbody>
        <tr>
          <td><input type="checkbox" v-model="showTileModel" /> 显示倾斜摄影模型</td>
        </tr>
        <tr>
          <td><input type="checkbox" v-model="showBuildingTip" /> 显示标签</td>
        </tr>
        <tr>
          <td><input type="checkbox" v-model="showBusLine" /> 显示公交线路</td>
        </tr>

      </tbody>
    </table>
  </div>
</template>

<script>
import {ref, watch } from 'vue';

export default {
  data() {
    return {
      showBusSchedule: false,
      imageScale: 1,
      minScale: 0.5,
      maxScale: 3,
      lastTouchDistance: 0,
      isDragging: false,
      dragStartX: 0,
      dragStartY: 0,
      imagePosition: {
        x: 0,
        y: 0
      },
      lastTouchCenter: { x: 0, y: 0 }
    }
  },

  setup(){
    // 响应式 有点学不懂  
    const showTileModel = ref(true);       
    const showBuildingTip = ref(true);  
    const showBusLine = ref(true);
    
    // 监视数据变化
    watch([showTileModel, showBuildingTip, showBusLine], () => {
      updateSceneShow();
    });

    function updateSceneShow() {
      // 添加存在性检查
      if (window.TileModel) {
        window.TileModel.show = showTileModel.value;
      }
      if (window.BuildingTip) {
        window.BuildingTip.show = showBuildingTip.value;
      }
      if (window.BusLineEntity) {
        window.BusLineEntity.show = showBusLine.value;
      }
    }
    return {
      showTileModel,
      showBuildingTip,
      showBusLine,
    };
  },

  mounted(){
    // const token = '002b7eb7dcf9fac7771e46f9220770b5' //天地图token
    // const tdtUrl = 'https://t{s}.tianditu.gov.cn/' //天地图url
    // const subdomains = ['0', '1', '2', '3', '4', '5', '6', '7'] // 不同地址加载图像？没看懂文档写的啥
    Cesium.Ion.defaultAccessToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiI0NjRmZWIzMi03YjEzLTQ5NGQtYWU4Ny02ZjkyN2IzZTM4YzQiLCJpZCI6MjkwOTYxLCJpYXQiOjE3NDM3ODYxNTJ9.9tCyS8-RvaImoVXrSE0ajjqSYRdaxpWm2M3vFlsre74"

    const viewer = new Cesium.Viewer("cesiumContainer", {
      animation: false, //右下角圆圈
      baseLayerPicker: true, // 地图切换控件
      scene3DOnly : true, //3d绘制
      fullscreenButton: true,
      geocoder: true,
      timeline: false,
      vrButton: false,
      homeButton: true, //主页按钮，研究一下设置主页位置
      infoBox: true,
      selectionIndicator: false,
      navigationHelpButton: false,
      navigationInstructionsInitiallyVisible:false,
      automaticallyTrackDataSourceClocks:false,
      sceneModePicker: false,
      terrain: Cesium.Terrain.fromWorldTerrain({requestVertexNormals:true}), //地形
      terrainExaggeration: 1.0, //地形缩放比
    });

/* ------------------------------------加载天地图------------------------------------- */
    // //图形：vec矢量图，img影像图，ter地形图，_c经纬度投影，_w球面墨卡托投影
    // viewer.imageryLayers.addImageryProvider(new Cesium.UrlTemplateImageryProvider({
    //   url: tdtUrl + 'DataServer?T=img_w&x={x}&y={y}&l={z}&tk=' + token,
    //   subdomains: subdomains,
    //   tilingScheme : new Cesium.WebMercatorTilingScheme(),
    //   maximumLevel : 18
    // })); 

    // //标注：cva矢量图，cia影像图，cta地形图，_c经纬度投影，_w球面墨卡托投影
    // viewer.imageryLayers.addImageryProvider(new Cesium.UrlTemplateImageryProvider({
    //   url: tdtUrl + 'DataServer?T=cia_w&x={x}&y={y}&l={z}&tk=' + token,
    //   subdomains: subdomains,
    //   tilingScheme : new Cesium.WebMercatorTilingScheme(),
    //   maximumLevel : 18
    // })); 

    // 国境线
    // viewer.imageryLayers.addImageryProvider(new Cesium.UrlTemplateImageryProvider({
    //   url: tdtUrl + 'DataServer?T=ibo_w&x={x}&y={y}&l={z}&tk=' + token,
    //   subdomains: subdomains,
    //   tilingScheme : new Cesium.WebMercatorTilingScheme(),
    //   maximumLevel : 10
    // })); 

    //初始定位到中国，和模型导入的定位会有冲突
    // viewer.camera.flyTo({
    //   destination: Cesium.Cartesian3.fromDegrees(103.84, 31.15, 17850000),
    // });


/* -----------------------------------渲染设置---------------------------------------- */
    // viewer.scene.globe.depthTestAgainstTerrain = true //地形深度测试，为什么又失败了啊啊啊啊啊啊啊
    // viewer.scene.globe.showGroundAtmosphere = true
    // viewer.scene.globe.enableLighting = true
    // viewer.scene.skyAtmosphere.show = true
    // viewer.scene.screenSpaceCameraController.enableCollisionDetection = true // 视角穿透？不理解
    // viewer.scene.highDynamicRange = true //HDR
    // // 抗锯齿
    // if (Cesium.FeatureDetection.supportsImageRenderingPixelated()) {
    //   viewer.resolutionScale = window.devicePixelRatio
    // }
    // viewer.scene.fxaa = true //开启快速近似抗锯齿
    // viewer.scene.postProcessStages.fxaa.enabled = true // 后处理开启抗锯齿


/* -----------------------------------tiaoshi---------------------------------------- */
    viewer.scene.debugShowFramesPerSecond = true;
    viewer.scene.debugShowGlobeDepth = true;
     window.viewer = viewer;
/* -----------------------------------主页按钮---------------------------------------- */
    // 包围球
    const boundingSphere = new Cesium.BoundingSphere(Cesium.Cartesian3.fromDegrees(120.0232087638, 30.2226337182, 30), 315.7837694);
    viewer.homeButton.viewModel.command.beforeExecute.addEventListener(cmd => {
      viewer.camera.flyToBoundingSphere(boundingSphere);
      cmd.cancel = true; // 阻止默认视角，重要
    });

    // 初始化场景
    this.initScene();
  },

  methods: {
    async initScene() {
      try {
        await this.LoadTileModel();
        await this.LoadBuildingTip();
        this.ShowPosition();
        await this.addWater();
        await this.addBusLine(); // 公交线路
      } catch (error) {
        console.error('场景初始化错误:', error);
      }
    },

    // async LoadSceneModel() { // 加载场景
    //   try {
    //     const tileset = await Cesium.Cesium3DTileset.fromIonAssetId(354307);//加载Ion中的资产=-美国-丹佛
    //     tileset.maximumScreenSpaceError = 2; //最大屏幕空间误差
        
    //     const translation = new Cesium.Cartesian3(0, -40, 0)
    //     tileset.modelMatrix = Cesium.Matrix4.fromTranslation(translation) //往上移动
        
    //     window.viewer.scene.primitives.add(tileset); //场景中添加模型
    //     window.SceneModel = tileset

    //     //window.viewer.zoomTo(tileset) //聚焦模型
    //   } catch (error) {
    //     alert(error)
    //   } 
    // },

    async LoadTileModel() { // 加载倾斜摄影模型
      try {
        // 本地
        //const tileset = await Cesium.Cesium3DTileset.fromUrl('/Data/tileset.json');
        // 远程阿里云
        const tileset = await Cesium.Cesium3DTileset.fromUrl('https://cesiumzust.oss-cn-hangzhou.aliyuncs.com/tileset.json');
        tileset.maximumScreenSpaceError = 16; //最大屏幕空间误差
        //backFaceCulling: false, //取消背面剔除

        // // 设置模型的位置，用经纬度
         tileset.modelMatrix = Cesium.Transforms.eastNorthUpToFixedFrame(
           Cesium.Cartesian3.fromDegrees(120.0117800036, 30.2113431735, 5)
          );
        // 等待完全加载
        await tileset.readyPromise;

        // 获取包围球中心坐标（ECEF）
        const boundingSphereCenter = tileset.boundingSphere.center;

        // 经纬度高程
        const cartographic = Cesium.Ellipsoid.WGS84.cartesianToCartographic(boundingSphereCenter);
        const longitude = Cesium.Math.toDegrees(cartographic.longitude);
        const latitude = Cesium.Math.toDegrees(cartographic.latitude);
        const height = cartographic.height;

        console.log('模型中心经纬度:', { longitude, latitude, height });

        // 可视化标记
        this.addCenterMarker(longitude, latitude, height);

    
        //const translation = new Cesium.Cartesian3(-50, 100, 70);
        //tileset.modelMatrix = Cesium.Matrix4.fromTranslation(translation);

        window.viewer.scene.primitives.add(tileset);
        window.TileModel = tileset;
        
        window.viewer.zoomTo(tileset) //聚焦模型
      } catch (error) {
        alert(error);
      } 
    },

    async LoadBuildingTip() { // 提示模型
      try {
        // 本地
        //const tileset = await Cesium.Cesium3DTileset.fromUrl('/Title/tileset.json');
        // 远程阿里云
        const tileset = await Cesium.Cesium3DTileset.fromUrl('https://cesiumtitle.oss-cn-hangzhou.aliyuncs.com/tileset.json');
        //tileset.maximumScreenSpaceError = 16; //最大屏幕空间误差
        //backFaceCulling: false, //取消背面剔除

        // 设置模型的位置，确保它出现在正确的经纬度上
          tileset.modelMatrix = Cesium.Transforms.eastNorthUpToFixedFrame(
            Cesium.Cartesian3.fromDegrees(120.0117800036, 30.2113431735, 2)
           );

        // // 等待模型完全加载
        //   await tileset.readyPromise;

        //   // 获取包围球中心坐标（ECEF）
        //   const boundingSphereCenter = tileset.boundingSphere.center;

        //   // 转换为经纬度高程
        //   const cartographic = Cesium.Ellipsoid.WGS84.cartesianToCartographic(boundingSphereCenter);
        //   const longitude = Cesium.Math.toDegrees(cartographic.longitude);
        //   const latitude = Cesium.Math.toDegrees(cartographic.latitude);
        //   const height = cartographic.height;

        //   console.log('title中心经纬度:', { longitude, latitude, height });

        //   // 添加可视化标记
        //   this.addCenterMarker(longitude, latitude, height);
    
        // const translation = new Cesium.Cartesian3(-50, 100, 70);
        // tileset.modelMatrix = Cesium.Matrix4.fromTranslation(translation);

        window.viewer.scene.primitives.add(tileset); //场景中添加模型
        window.BuildingTip = tileset; // 全局变量
        //window.viewer.zoomTo(tileset) //聚焦模型
      } catch (error) {
        alert(error);
      } 
    },
    /* -----------------------------------线路图---------------------------------------- */
    toggleBusSchedule() {
      this.showBusSchedule = true;
      this.resetImageView();
    },
    
    closeBusSchedule() {
      this.showBusSchedule = false;
      this.resetImageView();
    },
    
    resetImageView() {
      this.imageScale = 1;
      this.imagePosition = { x: 0, y: 0 };
      this.isDragging = false;
    },
    
    // 鼠标滚轮缩放
    handleWheel(event) {
      const rect = event.currentTarget.getBoundingClientRect()
      this.scaleImage(
        event.deltaY > 0 ? -0.1 : 0.1, // 缩放方向
        event.clientX - rect.left,     // 鼠标X坐标
        event.clientY - rect.top       // 鼠标Y坐标
      )
    },
    
    // 鼠标拖拽
    handleMouseDown(event) {
      this.isDragging = true
      this.dragStartX = event.clientX - this.imagePosition.x
      this.dragStartY = event.clientY - this.imagePosition.y
    },

    handleMouseMove(event) {
      if (!this.isDragging) return
      
      this.imagePosition.x = event.clientX - this.dragStartX
      this.imagePosition.y = event.clientY - this.dragStartY
      this.limitImagePosition()
    },
    
    handleMouseUp() {
      this.isDragging = false
    },
    
    // 触摸开始
    handleTouchStart(event) {
      if (event.touches.length === 1) {
        // 单指拖拽
        this.isDragging = true
        this.dragStartX = event.touches[0].clientX - this.imagePosition.x
        this.dragStartY = event.touches[0].clientY - this.imagePosition.y
      } else if (event.touches.length === 2) {
        // 双指缩放初始化
        const [t1, t2] = event.touches
        this.lastTouchDistance = Math.hypot(t2.clientX - t1.clientX, t2.clientY - t1.clientY)
        this.lastTouchCenter = {
          x: (t1.clientX + t2.clientX) / 2,
          y: (t1.clientY + t2.clientY) / 2
        }
      }
    },

    
    // 触摸移动
    handleTouchMove(event) {
      if (event.touches.length === 1 && this.isDragging) {
        // 单指拖拽
        this.imagePosition.x = event.touches[0].clientX - this.dragStartX
        this.imagePosition.y = event.touches[0].clientY - this.dragStartY
        this.limitImagePosition()
      } else if (event.touches.length === 2) {
        // 双指缩放
        const [t1, t2] = event.touches
        const rect = event.currentTarget.getBoundingClientRect()
        
        // 计算中心点坐标
        const centerX = (t1.clientX + t2.clientX) / 2 - rect.left
        const centerY = (t1.clientY + t2.clientY) / 2 - rect.top
        
        // 计算缩放量
        const currentDistance = Math.hypot(t2.clientX - t1.clientX, t2.clientY - t1.clientY)
        const delta = (currentDistance - this.lastTouchDistance) / 100
        
        this.scaleImage(delta, centerX, centerY)
        this.lastTouchDistance = currentDistance
      }
    },

    // 触摸结束
    handleTouchEnd(event) {
      if (event.touches.length === 0) {
        this.isDragging = false
        this.lastTouchDistance = 0
      }
    },
    
    // 核心缩放方法
    scaleImage(delta, centerX, centerY) {
      const oldScale = this.imageScale
      const newScale = Math.min(this.maxScale, Math.max(this.minScale, this.imageScale + delta))

      if (newScale === oldScale) return

      // 计算缩放中心偏移
      const scaleFactor = newScale / oldScale
      this.imagePosition.x = centerX - (centerX - this.imagePosition.x) * scaleFactor
      this.imagePosition.y = centerY - (centerY - this.imagePosition.y) * scaleFactor
      
      this.imageScale = newScale
      this.limitImagePosition()
    },

    // 位置限制方法
    limitImagePosition() {
      if (!this.$refs.scheduleContainer || !this.$refs.scheduleImage) return;
      
      const containerRect = this.$refs.scheduleContainer.getBoundingClientRect();
      const imgWidth = this.$refs.scheduleImage.naturalWidth * this.imageScale;
      const imgHeight = this.$refs.scheduleImage.naturalHeight * this.imageScale;

      const maxX = (imgWidth - containerRect.width) * 0.5;
      const minX = -maxX;
      const maxY = (imgHeight - containerRect.height) * 0.5;
      const minY = -maxY;

      this.imagePosition.x = Math.min(maxX, Math.max(minX, this.imagePosition.x));
      this.imagePosition.y = Math.min(maxY, Math.max(minY, this.imagePosition.y));
    },

    addCenterMarker(lon, lat, height) {
      const viewer = window.viewer;
      
      // 红色点标记
      viewer.entities.add({
        position: Cesium.Cartesian3.fromDegrees(lon, lat, height),
        point: {
          color: Cesium.Color.RED,
          pixelSize: 10,
          outlineColor: Cesium.Color.WHITE,
          outlineWidth: 2
        },
        label: {
          text: `模型中心点\n经度: ${lon.toFixed(6)}°\n纬度: ${lat.toFixed(6)}°\n高程: ${height.toFixed(2)}米`,
          font: '14px sans-serif',
          fillColor: Cesium.Color.WHITE,
          backgroundColor: Cesium.Color.fromCssColorString('#00000080'),
          showBackground: true,
          verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
          pixelOffset: new Cesium.Cartesian2(0, -20)
        }
      });
    },

    ShowPosition() { // 鼠标悬停经纬度
      const handler = new Cesium.ScreenSpaceEventHandler(); // 监听窗口的左键点击事件
      handler.setInputAction(function (movement) {
        const ray = window.viewer.camera.getPickRay(movement.endPosition);
        const position = window.viewer.scene.globe.pick(ray, window.viewer.scene);
        
        if (Cesium.defined(position)) {
            const cartographic = Cesium.Cartographic.fromCartesian(position);
            
            const longitude = Cesium.Math.toDegrees(cartographic.longitude).toFixed(6);
            const latitude = Cesium.Math.toDegrees(cartographic.latitude).toFixed(6);
            const height = cartographic.height.toFixed(2);

            document.getElementById('showCoordinates').innerText = `${longitude} , ${latitude} , ${height}`;
        }
      },
      Cesium.ScreenSpaceEventType.MOUSE_MOVE)
    },

    addWater() { // 水体
      try {
        const waterMaterial = new Cesium.Material({
          fabric: {
            type: 'Water',
            uniforms: {
              baseWaterColor: new Cesium.Color(0.0, 0.25, 0.3, 0.9), // 水体颜色
              normalMap: '/Textures/norm0001.png', // 法线贴图
              frequency: 100, // 水面波纹频率 
              animationSpeed: 0.001, // 水面波动动画速率
              amplitude: 20, // 水面波动幅度
              specularIntensity: 2.0, // 水面反射强度
            }
          }
        });

        const waterKeyPoint = [// 水体关键点数组

          120.016250, 30.218375,
          120.016592, 30.218233,
          120.017746, 30.218113,
          120.019161, 30.218730,
          120.020083, 30.219830,
          120.020574, 30.220600,
          120.020675, 30.221526,
          120.024671, 30.223876,
          120.024462, 30.224050,
          120.021604, 30.222518,
          120.018557, 30.221092
        ];

        const waterPolygon = new Cesium.PolygonGeometry({ // 创建水体几何体
          polygonHierarchy: new Cesium.PolygonHierarchy(
            Cesium.Cartesian3.fromDegreesArray(waterKeyPoint)
          ), 
          extrudedHeight: 32.0, // 水体底高程
          height: 10.0, // 水体高度
        });

        const waterPrimitive = new Cesium.Primitive({
          geometryInstances: new Cesium.GeometryInstance({
            geometry: waterPolygon
          }),
          appearance: new Cesium.EllipsoidSurfaceAppearance({
            material: waterMaterial
          }),
          shadows: Cesium.ShadowMode.RECEIVE_ONLY
        });

        window.viewer.scene.primitives.add(waterPrimitive);
        console.log('水体添加成功');
      } catch (error) {
        console.error('添加水体时发生错误:', error);
      }
    },

    async addBusLine() { // 添加公交线路
      try {
        const viewer = window.viewer;
        
        // 公交线路数据：每个子数组代表一条路线
        const busRoutes = [
          // 路线1：经过的点坐标（经度，纬度）
          [
            { lng: 120.016720, lat: 30.218084, isStation: true, name: "西和公寓北" },
            { lng: 120.015690, lat: 30.218662, isStation: false },
            { lng: 120.015825, lat: 30.218742, isStation: false },
            { lng: 120.016158, lat: 30.218602, isStation: false },
            { lng: 120.016374, lat: 30.218726, isStation: false },
            { lng: 120.017340, lat: 30.219486, isStation: true, name: "田径场" },
            { lng: 120.017993, lat: 30.220370, isStation: false },
            { lng: 120.018765, lat: 30.221463, isStation: false },
            { lng: 120.018640, lat: 30.222297, isStation: false },
            { lng: 120.019403, lat: 30.222741, isStation: false },
            { lng: 120.020043, lat: 30.222888, isStation: false },
            { lng: 120.019566, lat: 30.223291, isStation: true, name: "师生服务中心" },
            { lng: 120.017788, lat: 30.225626, isStation: true, name: "闻理园A2"  },
            { lng: 120.018188, lat: 30.226244, isStation: false },
            { lng: 120.018823, lat: 30.226379, isStation: false },
            { lng: 120.019507, lat: 30.226229, isStation: true, name: "闻理园A4"  },
            { lng: 120.020229, lat: 30.226133, isStation: false },
            { lng: 120.020680, lat: 30.226310, isStation: false },
            { lng: 120.021502, lat: 30.226088, isStation: false },
            { lng: 120.021927, lat: 30.225865, isStation: true, name: "精艺园"  },
            { lng: 120.022569, lat: 30.225520, isStation: false },
            { lng: 120.022946, lat: 30.225065, isStation: false },
            { lng: 120.023085, lat: 30.224751, isStation: false }, // 产教中心门口
            { lng: 120.022787, lat: 30.224088, isStation: true, name: "实验大楼"  },
            { lng: 120.022202, lat: 30.223783, isStation: false },
            { lng: 120.022085, lat: 30.222704, isStation: false },
            { lng: 120.022342, lat: 30.222328, isStation: false }, // 过桥
            { lng: 120.021735, lat: 30.222063, isStation: true, name: "图书馆"  },
            { lng: 120.021173, lat: 30.221635, isStation: false },
            { lng: 120.022058, lat: 30.221139, isStation: true, name: "行政楼"  },
            { lng: 120.022922, lat: 30.221355, isStation: false },
            { lng: 120.023164, lat: 30.220989, isStation: false },
            { lng: 120.022542, lat: 30.220664, isStation: true, name: "浙江科技大学公交站"  },
            { lng: 120.022198, lat: 30.220375, isStation: false },
            { lng: 120.020300, lat: 30.218638, isStation: false },
            { lng: 120.019203, lat: 30.217584, isStation: true, name: "冲天庙公交站"  },
            { lng: 120.018420, lat: 30.216674, isStation: false },
            { lng: 120.018049, lat: 30.217372, isStation: false },
            { lng: 120.017634, lat: 30.217989, isStation: false },
            { lng: 120.016720, lat: 30.218084, isStation: true, name: "西和公寓北" },
          ],
          // 路线2
          [
            { lng: 120.026119, lat: 30.220649, isStation: true, name: "起始站" },
            { lng: 120.026189, lat: 30.220951, isStation: false },
            { lng: 120.027215, lat: 30.220971, isStation: true, name: "东和公寓东" },
            { lng: 120.027566, lat: 30.220987, isStation: false },
            { lng: 120.026827, lat: 30.222680, isStation: false },
            { lng: 120.025873, lat: 30.223878, isStation: true, name: "习得园" },
            { lng: 120.025620, lat: 30.224155, isStation: false },
            { lng: 120.025226, lat: 30.224032, isStation: false },
            { lng: 120.024276, lat: 30.223598, isStation: false },
            { lng: 120.022506, lat: 30.222472, isStation: true, name: "图书馆" },
            { lng: 120.022364, lat: 30.222398, isStation: false },
            { lng: 120.022020, lat: 30.222778, isStation: false },
            { lng: 120.020063, lat: 30.222879, isStation: false },
            { lng: 120.019586, lat: 30.223311, isStation: true, name: " " }, // 师生服务中心
            { lng: 120.017808, lat: 30.225646, isStation: true, name: " "  }, // 闻理园A2
            { lng: 120.018843, lat: 30.226399, isStation: false },
            { lng: 120.019527, lat: 30.226249, isStation: true, name: " "  }, // 闻理园A4
            { lng: 120.020249, lat: 30.226153, isStation: false },
            { lng: 120.020700, lat: 30.226330, isStation: false },
            { lng: 120.021522, lat: 30.226108, isStation: false },
            { lng: 120.021947, lat: 30.225885, isStation: true, name: " "  }, // 精艺园
            { lng: 120.022589, lat: 30.225540, isStation: false },
            { lng: 120.022966, lat: 30.225085, isStation: false },
            { lng: 120.023105, lat: 30.224771, isStation: false }, // 产教中心门口
            { lng: 120.022807, lat: 30.224108, isStation: true, name: " "  }, // 实验大楼
            { lng: 120.022222, lat: 30.223803, isStation: false },
            { lng: 120.022105, lat: 30.222724, isStation: false },
            { lng: 120.022402, lat: 30.222438, isStation: false }, // 过桥
            
            { lng: 120.022566, lat: 30.222532, isStation: true, name: " " }, // 图书馆
            { lng: 120.024316, lat: 30.223638, isStation: false },
            { lng: 120.025266, lat: 30.224072, isStation: false },
            { lng: 120.025640, lat: 30.224195, isStation: false },
            { lng: 120.025913, lat: 30.223918, isStation: true, name: " " },  // 习得园
            { lng: 120.026867, lat: 30.222720, isStation: false },
            { lng: 120.027606, lat: 30.220967, isStation: false },
            { lng: 120.027275, lat: 30.220951, isStation: true, name: " " }, // 东和公寓东
            { lng: 120.026669, lat: 30.220960, isStation: false },
            { lng: 120.026449, lat: 30.220207, isStation: false },
            { lng: 120.026126, lat: 30.220220, isStation: false },
            { lng: 120.026119, lat: 30.220649, isStation: true, name: " " }, // 起始站
          ]
        ];
        
        // 创建实体集合
        const busLineEntityCollection = new Cesium.CustomDataSource("公交线路");
        
        // 遍历每条路线
        busRoutes.forEach((route, routeIndex) => {
          // 创建点的笛卡尔坐标数组
          const positions = [];
          
          // 处理路线上的每个点
          route.forEach((point, pointIndex) => {
            // 转换为笛卡尔坐标并添加到数组
            const position = Cesium.Cartesian3.fromDegrees(point.lng, point.lat, 80);
            positions.push(position);
            
            // 如果是站点，添加标记
            if (point.isStation) {
              busLineEntityCollection.entities.add({
                position: position,
                name: point.name || `站点${pointIndex}`,
                point: {
                  pixelSize: 12,
                  color: Cesium.Color.RED,
                  outlineColor: Cesium.Color.WHITE,
                  outlineWidth: 2,
                  // heightReference: Cesium.HeightReference.RELATIVE_TO_GROUND
                },
                label: {
                  text: point.name || `站点${pointIndex}`,
                  font: '14px sans-serif',
                  fillColor: Cesium.Color.WHITE,
                  style: Cesium.LabelStyle.FILL_AND_OUTLINE,
                  outlineWidth: 2,
                  outlineColor: Cesium.Color.BLACK,
                  verticalOrigin: Cesium.VerticalOrigin.BOTTOM,
                  pixelOffset: new Cesium.Cartesian2(0, -16),
                  // heightReference: Cesium.HeightReference.RELATIVE_TO_GROUND,
                  distanceDisplayCondition: new Cesium.DistanceDisplayCondition(0, 5000),
                  show: true
                }
              });
            }
          });
          
          // 流动线
          if (positions.length >= 2) {
            const lineColor = routeIndex === 0 ? 
              new Cesium.Color(0, 0.6, 1, 1.0) : // 路线1颜色
              new Cesium.Color(1, 0.5, 0, 1.0);  // 路线2颜色
            
            busLineEntityCollection.entities.add({
              name: `公交路线${routeIndex+1}`,
              polyline: {
                positions: positions,
                width: 8,
                material: new Cesium.PolylineGlowMaterialProperty({
                  glowPower: 0.2,
                  color: lineColor
                }),
                // heightReference: Cesium.HeightReference.RELATIVE_TO_GROUND
              }
            });
            
            //模拟流动效果
            const pointEntity = busLineEntityCollection.entities.add({
              position: positions[0],
              point: {
                pixelSize: 12,
                color: lineColor,
                outlineColor: Cesium.Color.WHITE,
                outlineWidth: 2,
                // heightReference: Cesium.HeightReference.RELATIVE_TO_GROUND
              }
            });
            
            // 流动点动画
            let animationStartTime = Cesium.JulianDate.now();
            const durationSeconds = 60; // 完整循环所需时间
    
            viewer.scene.preRender.addEventListener(function() {
              if (pointEntity && pointEntity.position) {
                const currentTime = Cesium.JulianDate.now();
                const elapsedSeconds = Cesium.JulianDate.secondsDifference(currentTime, animationStartTime);
                const t = (elapsedSeconds % durationSeconds) / durationSeconds;
                
                let currentPos;
                const segmentCount = positions.length - 1;
                const segmentIndex = Math.floor(t * segmentCount);
                const segmentT = (t * segmentCount) % 1;
                
                if (segmentIndex < segmentCount) {
                  currentPos = Cesium.Cartesian3.lerp(
                    positions[segmentIndex],
                    positions[segmentIndex + 1],
                    segmentT,
                    new Cesium.Cartesian3()
                  );
                } else {
                  currentPos = positions[positions.length - 1];
                }
                pointEntity.position = currentPos;
              }
            });
          }
        });
        
        viewer.dataSources.add(busLineEntityCollection);
        window.BusLineEntity = busLineEntityCollection;
        console.log('公交线路添加成功');
      } catch (error) {
        console.error('添加公交线路时发生错误:', error);
      }
    }

  },
}

</script>


<style scoped>
.hello {
  width: 100%;
  height: 100%;
}
#cesiumContainer {
  position: relative;
  width: 100%;
  height: 100vh;
  margin: 0;
  padding: 0;
  overflow: hidden;
}

#sceneOption {
  position: absolute; 
  top: 120px; 
  left: 10px; 
}

/* 公交按钮样式 */
.bus-button {
  position: absolute;
  bottom: 20px;
  right: 20px;
  width: 50px;
  height: 50px;
  background-color: #fff;
  border-radius: 50%;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  z-index: 1000;
  transition: transform 0.2s;
}

.bus-button:hover {
  transform: scale(1.1);
}

.bus-button img {
  width: 30px;
  height: 30px;
}

/* 公交时刻表弹窗样式 */
.bus-schedule-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000;
}

.bus-schedule-container {
  position: relative;
  width: 80%;
  height: 80%;
  background-color: #fff;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.5);
}

.close-button {
  position: absolute;
  top: 10px;
  left: 10px;
  width: 30px;
  height: 30px;
  background-color: #ff4d4f;
  color: white;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 20px;
  cursor: pointer;
  z-index: 2010;
}

.schedule-image-container {
  touch-action: none;
  overflow: hidden;
}

.schedule-image-container img {
  transition: transform 0.15s ease-out;
}

#showCoordinates {
  position: absolute; /* 左下 */
  bottom: 25px;
  left: 10px;
}
.labels {
  background-color: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 5px; 
  border-radius: 5px; 
  font-size: 18px; 
  z-index: 1000; 
}
</style>
