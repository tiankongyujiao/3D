<template>
<div id="app">
  <div id="nav" style="width: 600px; height: 600px; position: absolute; left: 200px; top: 200px;">

  </div>
</div>
</template>

<script>
import * as THREE from 'three';
import {
  GLTFLoader
} from 'three/examples/jsm/loaders/GLTFLoader.js';
import {
  FBXLoader
} from 'three/examples/jsm/loaders/FBXLoader.js';
import {
  DRACOLoader
} from 'three/examples/jsm/loaders/DRACOLoader.js';
import {
  OrbitControls
} from 'three/examples/jsm/controls/OrbitControls.js';

export default {
  name: 'Home',
  components: {

  },
  data() {
    return {
      clock: null,
      render: null,
      camera: null,
      scene: null,
      loader: null,
      mixer: null,
      mesh: null,
      group: null,
      width: window.innerWidth,
      height: window.innerHeight,
      clickName: ''
    }
  },
  mounted() {
    var _this = this;
    this.init();
    this.loaderGlft();
    this.renderer.render(this.scene, this.camera);

    let controls = new OrbitControls(_this.camera, _this.renderer.domElement);
    controls.addEventListener('change', function () {
      _this.renderer.render(_this.scene, _this.camera);
    });
    // use if there is no animation loop
    controls.minDistance = 2;
    controls.maxDistance = 10;
    controls.target.set(0, 0, -0.2);
    controls.update();
  },
  methods: {
    init() {
      const _this = this;
      // 渲染器
      this.renderer = new THREE.WebGLRenderer({
        alpha: true,
        antialias: true
      });
      var width = document.getElementById('nav').getBoundingClientRect().width;
      var height = document.getElementById('nav').getBoundingClientRect().height;
      this.renderer.physicallyCorrectLights = true; // Boolean 默认false ，物理的正确的照明方式
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setSize(width, height);
      document.getElementById('nav').appendChild(this.renderer.domElement);

      this.camera = new THREE.PerspectiveCamera(45, width / height, 0.01, 1000);
      this.camera.position.set(118, 60, 227);

      this.scene = new THREE.Scene();
      console.log(_this.scene.position, '_this.scene.position')
      this.camera.lookAt(_this.scene.position);

      // 初始化光源
      const hemisphereLight = new THREE.HemisphereLight(0xffffbb, 0x080820, 1)
      this.scene.add(hemisphereLight)
      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8 * Math.PI)
      directionalLight.position.set(1, 0.5, 1)
      this.scene.add(directionalLight)
      const ambientLight = new THREE.AmbientLight(0xffffff, 0.1)
      this.scene.add(ambientLight)
      var light = new THREE.PointLight(0xff0000, 1, 100);
      light.position.set(150, 50, 50);
      this.scene.add(light);

      window.addEventListener('click', this.onMouseClick, false);
    },
    // 鼠标点击
    onMouseClick(event) {
      var _this = this;
      var raycaster = new THREE.Raycaster()
      var mouse = new THREE.Vector2()

      //将鼠标点击位置的屏幕坐标转换成threejs中的标准坐标
      mouse.x = ((event.clientX - _this.renderer.domElement.getBoundingClientRect().left) / _this.renderer.domElement.offsetWidth) * 2 - 1; // 标准设备横坐标
      mouse.y = -((event.clientY - _this.renderer.domElement.getBoundingClientRect().top) / _this.renderer.domElement.offsetHeight) * 2 + 1; // 标准设备纵坐标

      // console.log(mouse.x, mouse.y, event.clientX / window.innerWidth, event.clientY / window.innerHeight);
      // 通过鼠标点的位置和当前相机的矩阵计算出raycaster
      raycaster.setFromCamera(mouse, _this.camera);
      console.log(_this.scene, 'this.scene')
      // 获取raycaster直线和所有模型相交的数组集合

      var intersects = raycaster.intersectObjects(_this.scene.children, true); // [5].children[0].children[0].children
      console.log(intersects);

      if (intersects.length > 0) {
        _this.clickName = intersects[0].object.name;
        console.log(_this.clickName, 'clickName')
      }
    },
    // 加载glft模型
    loaderGlft() {
      var _this = this;
      this.loader = new GLTFLoader();

      var dracoLoader = new DRACOLoader();
      dracoLoader.setDecoderPath('/draco/');
      this.loader.setDRACOLoader(dracoLoader);

      // this.loader.load('/01tex.gltf', function (gltf) {
      // this.loader.load('/glTF/DamagedHelmet.gltf', function ( gltf ) {
      // this.loader.load('/matilda/scene.gltf', function ( gltf ) {
      this.loader.load('/amei/amei.gltf', function (gltf) {
        console.log(gltf, 'gltf');

        const loadscene = gltf.scene;
        loadscene.scale.set(1, 1, 1);
        // 矫正位置，模型自适应大小
        _this.setPositionAndAdd(loadscene);

        _this.renderer.render(_this.scene, _this.camera);
      });
    },
    // 矫正位置, 模型自适应大小
    setPositionAndAdd(loadscene) {
      var _this = this;
      var group = new THREE.Group();
      group.add(loadscene);

      let bbox = new THREE.Box3().setFromObject(group);
      let mdlen = bbox.max.x - bbox.min.x; //边界的最小坐标值 边界的最大坐标值
      let mdhei = bbox.max.y - bbox.min.y;
      let mdwid = bbox.max.z - bbox.min.z;

      group.position.set(0, 0, 0);

      let dist = Math.abs(_this.camera.position.z - group.position.z - (mdwid / 2));
      //console.log('dist值为:' + dist );
      let vFov = _this.camera.fov * Math.PI / 180;
      //console.log('vFov值为:' + vFov );
      let vheight = 2 * Math.tan(vFov * 0.5) * dist;
      //console.log('vheight值为:' + vheight );
      let fraction = mdhei / vheight;
      // console.log('fraction值为:' + fraction );
      let finalHeight = _this.height * fraction;
      //console.log('finalHeight值为:' + finalHeight);
      let finalWidth = (finalHeight * mdlen) / mdhei;
      //console.log('finalWidth值为:' + finalWidth );                

      let value1 = _this.width / finalWidth;
      console.log('value1缩放比例值为:' + value1);
      let value2 = _this.height / finalHeight;
      console.log('value2缩放比例值为:' + value2);

      if (value1 >= value2) {
        group.scale.set(value2, value2, value2);
      } else {
        group.scale.set(value1, value1, value1);
      }
      let bbox2 = new THREE.Box3().setFromObject(group)
      let mdlen2 = bbox2.max.x - bbox2.min.x;
      let mdhei2 = bbox2.max.y - bbox2.min.y;
      let mdwid2 = bbox2.max.z - bbox2.min.z;
      group.position.set(-(bbox2.max.x + bbox2.min.x) / 2,
        -(bbox2.max.y + bbox2.min.y) / 2,
        -(bbox2.max.z + bbox2.min.z) / 2 - (bbox2.max.z - bbox2.min.z) / 2);

      _this.scene.add(group);
    }
  }
}
</script>

<style lang="less">
body {
  margin: 0;
  overflow: hidden;
}
</style>
