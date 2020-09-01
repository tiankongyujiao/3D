# test

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).

1. 3d模型gltf格式导入，gltf是使用draco压缩过以后的模型，所以需要引入DRACOLoader  

2. 在使用Raycaster计算点击的物体的时候，鼠标点击的点计算不是简单的：
```
mouse.x = ( event.clientX / window.innerWidth ) * 2 - 1;
mouse.y = - ( event.clientY / window.innerHeight ) * 2 + 1;
```
而是要（此时canvas大小和窗口大小不一样）：
```
  //将鼠标点击位置的屏幕坐标转换成threejs中的标准坐标
  mouse.x = ((event.clientX - _this.renderer.domElement.getBoundingClientRect().left) / _this.renderer.domElement.offsetWidth) * 2 - 1; // 标准设备横坐标
  mouse.y = -((event.clientY - _this.renderer.domElement.getBoundingClientRect().top) / _this.renderer.domElement.offsetHeight) * 2 + 1; // 标准设备纵坐标
```

3. 使用 **setPositionAndAdd** 方法，把载入的模型自适应大小  