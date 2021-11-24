#### GLTFLoader
https://blog.csdn.net/weixin_40045529/article/details/108666431
https://www.jianshu.com/p/8265d5148a5b


#### THREE.PMREMGenerator
这个类从一个cubeMap环境纹理生成一个PMREM.这允许基于材质粗糙度快速访问不同级别的模糊。通过这种方式我们可以在限制采样计算的同时保持平滑的漫射照明的分辨率。
- 可以解决环境照明问题。
-  pmremGenerator.fromScene
从提供的场景生成PMREM，如果网络带宽低，这可能比使用图像更快。
```
// 使用hdr作为背景色
new THREE.PMREMGenerator( this.renderer );
```

#### OrbitControls
- 可以对场景进行缩放，平移，旋转操作。本质上改变的并不是场景，而是相机的参数。
```
const controls = new OrbitControls(this.camera, this.renderer.domElement)
controls.enablePan = false; //禁止右键拖拽
controls.enableZoom = false;//禁止缩放
controls.enableRotate = false; //禁止旋转
// 缩放范围
controls.minZoom = 0.5;
controls.maxZoom = 2;
// 上下旋转范围
controls.minPolarAngle = 0;
controls.maxPolarAngle = Math.PI;
// 左右旋转范围
controls.minAzimuthAngle = -Math.PI * (100 / 180);
controls.maxAzimuthAngle = Math.PI * (100 / 180);
//监听鼠标事件，触发渲染函数，更新canvas画布渲染效果
controls.addEventListener('change', render);
```
