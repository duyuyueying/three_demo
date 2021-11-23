#### THREE.AnimationMixer 混合器
- 该构造函数接收一个一个模型对象，实例可操作模型对象中所有自对象的帧动画
```
const mesh = ....
const mixer = new THREE.AnimationMixer(mesh)
```
- this.mixer.clipAction()
clip作为参数。返回一个操作对象AnimationAction
```
// 多个帧动画作为元素创建一个剪辑clip对象，命名"default"，持续时间20
var clip = new THREE.AnimationClip("default", duration, [posTrack, colorKF, scaleTrack]);
const animationAction = mixer.clipAction(clip)
// 调节播放速度
animationAction.timeScale = 20
// 开始播放
animationAction.play()
// 不循环播放
animationAction.loop = THREE.LoopOnce;
// 暂停在最后一针播放的状态
animationAction.clampWhenFinished = true
// 设置开始播放的时间
animationAction.time = 10
// 设置播放结束的时间
clip.duration = 18
```
- 渲染动画
```
var clock = new THREE.Clock()
function render() {
  renderer.render(scene, camera)
  requestAnimationFrame(render)
  // 获得两帧的时间间隔
  let mixerUpdateDelta = clock.getDelta()
  // 更新混合器相关时间
  mixer.update(mixerUpdateDelta)
}
```


#### this.camera.lookAt(0,1,0)
调整相机的朝向

#### 雾化效果
也就是场景中越远的位置看起来越模糊
参数一表示场景中雾的颜色
参数二表示受雾化影响的最近距离(以相机位置为准)
参数三表示受雾化影响的最远距离(以相机位置为准)
```
scene.fog = new THREE.Fog(0xcc0000, 1, 1000);
```

#### 设备像素比
```
this.renderer.setPixelRatio(window.devicePixelRatio)
```

#### 使用state管理
```
this.stats = new Stats();
container.appendChild( this.stats.dom );
function render() {
  this.stats.update()
}
```

#### 环境光THREE.HemisphereLight
- 创建更加贴近自然的户外光照效果
- 不会产生阴影
- 属性介绍：color: 从天空中发出的光线的颜色
- 属性介绍：groundColor: 从地面发出的光线的颜色
- 属性介绍：intensity: 光源照射的强度。默认值为：1。
- 属性介绍：position: 光源在场景中的位置。默认值为：(0, 100, 0)
- 属性介绍：visible: 设为 ture（默认值），光源就会打开。设为 false，光源就会关闭。

#### 平行光 THREE.DirectionLight
- 可以看作是距离很远的光，它发出的所有光线都是相互平行的
- 被平行光照亮的整个区域接受到的光强是一样的。
- castShadow：投影。如果设置为 true，这个光源就会生成阴影
- onlyShadow：仅阴影。如果此属性设置为 true，则该光源只生成阴影，而不会在场景中添加任何光照。
- shadow.camera.near：投影近点。表示距离光源的哪一个位置开始生成阴影。
- shadow.camera.far：投影远点。表示到距离光源的哪一个位置可以生成阴影。
- shadow.camera.left：投影左边界。
- shadow.camera.right：投影右边界。
- shadow.camera.top：投影上边界。
- shadow.camera.bottom：投影下边界。

#### 显示骨架
```
this.skeleton = new THREE.SkeletonHelper(this.model)
this.skeleton.visible = false;
this.scene.add(this.skeleton)
```



