<template>
<div id="app">
  <div id="nav" style="width: 600px; height: 600px;">
    <img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mP8/5+hHgAHggJ/PchI7wAAAABJRU5ErkJggg==" />
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
      height: window.innerHeight
    }
  },
  mounted() {
    var _this = this;
    this.init();
    this.loaderWlft();
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
      this.renderer = new THREE.WebGLRenderer({
        alpha: true,
        antialias: true
      });
      this.renderer.physicallyCorrectLights = true
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      document.getElementById('nav').appendChild(this.renderer.domElement);

      this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.01, 1000);
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

      // var geometry = new THREE.BoxGeometry(10, 10, 10);; //球体

      // var material = new THREE.MeshPhongMaterial({
      //   color: 0xff0000,
      //   specular: 0x444444, //高光部分的颜色
      //   shininess: 20, //高光部分的亮度，默认30
      // });
      // var mesh = new THREE.Mesh(geometry, material);
      // this.scene.add(mesh);

      // 辅助三维坐标系AxisHelper
      var axisHelper = new THREE.AxesHelper(250);
      this.scene.add(axisHelper);

      var raycaster = new THREE.Raycaster()
      var mouse = new THREE.Vector2()

      function onMouseClick(event) {
        console.log(123)
        //将鼠标点击位置的屏幕坐标转换成threejs中的标准坐标
        mouse.x = (event.clientX / window.innerWidth) * 2 - 1
        mouse.y = (event.clientY / window.innerHeight) * 2 + 1

        // 通过鼠标点的位置和当前相机的矩阵计算出raycaster
        raycaster.setFromCamera(mouse, _this.camera);
        console.log(_this.scene, 'this.scene')
        // 获取raycaster直线和所有模型相交的数组集合

        var intersects = raycaster.intersectObjects(_this.scene.children, true); // [5].children[0].children[0].children
        console.log(intersects);

        //将所有的相交的模型的颜色设置为红色
        // for (var i = 0; i < intersects.length; i++) {
        //   intersects[i].object.material.color.set(0xff0000);
        // }

      }
      window.addEventListener('click', onMouseClick, false);
    },
    loaderWlft() {
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

        var group = new THREE.Group();
        group.add(loadscene);
        // _this.scene.add(group);

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
          // group.scale.set(value2, value2, value2);
        } else {
          // group.scale.set(value1, value1, value1);
        }
        let bbox2 = new THREE.Box3().setFromObject(group)
        let mdlen2 = bbox2.max.x - bbox2.min.x;
        let mdhei2 = bbox2.max.y - bbox2.min.y;
        let mdwid2 = bbox2.max.z - bbox2.min.z;
        group.position.set(-(bbox2.max.x + bbox2.min.x) / 2,
          -(bbox2.max.y + bbox2.min.y) / 2,
          -(bbox2.max.z + bbox2.min.z) / 2 - (bbox2.max.z - bbox2.min.z) / 2);

        _this.scene.add(group);

        // let boxhelper = new THREE.BoxHelper(group, 0xbe1915); //外面红色框
        // _this.scene.add(boxhelper);

        // gltf.scene.translateY(-1.6);
        // gltf.scene.scale.set(0.05, 0.05, 0.05)
        // var i = 0;
        // gltf.scene.traverse(function (child) {
        //   i++;
        //   console.log(child, 'child', child.isMesh)
        //   if (child.isMesh) {

        //     // TOFIX RoughnessMipmapper seems to be broken with WebGL 2.0
        //     // roughnessMipmapper.generateMipmaps( child.material );

        //   }

        // });
        // console.log(i, 'i')
        _this.renderer.render(_this.scene, _this.camera);
      });

      // var loader = new FBXLoader();//创建一个FBX加载器
      // loader.load("/kongtiao.fbx", function(obj) {
      // 	console.log(obj);//查看加载后返回的模型对象
      // 	_this.scene.add(obj);
      // 	// 适当平移fbx模型位置
      // 	obj.position.set(4800, 2924, 3300);
      // 	obj.scale.set(0.9, 0.9, 0.9)
      // 	// obj.scale.set(0.5,0.5,0.5)
      // 	console.log(_this.scene, 'this.scene', _this.scene.scale)
      // 	_this.renderer.render( _this.scene, _this.camera );
      // })
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
