<template>
  <div id="container"></div>
</template>

<script>
import * as THREE from 'three'
import Stats from 'three/examples/jsm/libs/stats.module.js';
import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls.js';
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader.js'
import { DRACOLoader } from 'three/examples/jsm/loaders/DRACOLoader.js';
export default {
  data(){
    return {
      camera: null,
      scene: null,
      renderer: null,
      width: window.innerWidth,
      height: window.innerHeight,
      idleAction: null,
      runAction: null,
      walkAction: null,
      actions: [],
      setting: {
          'show model': true,
          'show skeleton': false,
          'deactivate all': this.deactivateAllActions,
          'activate all': this.activateAllActions,
          'pause/continue': this.pauseContinue,
          'make single step': this.toSingleStepMode,
          'modify step size': 0.05,
          'from walk to idle': ()=> {
              this.prepareCrossFade( this.walkAction, this.idleAction, 1.0 );
          },
          'from idle to walk':()=> {

              this.prepareCrossFade( this.idleAction, this.walkAction, 0.5 );

          },
          'from walk to run': () => {
              this.prepareCrossFade( this.walkAction, this.runAction, 2.5 );
          },
          'from run to walk': ()=> {
              this.prepareCrossFade( this.runAction, this.walkAction, 5.0 );
          },
          'use default duration': true,
          'set custom duration': 3.5,
          'modify idle weight': 0.0,
          'modify walk weight': 1.0,
          'modify run weight': 0.0,
          'modify time scale': 1.0
      },
      clock: null,
      stats: null,
      model: null, // 模型
      skeleton: null, // 骨架
      singleStepMode: false, // 是否是单步模式
      sizeOfNextStep: 0,
      crossFadeControls: [],
      startAction: null
    }
  },
  mounted() {
      this.init()
  },
  methods: {
    init() {
      this.clock = new THREE.Clock();
      this.setCamera()
      this.setScene()
      this.setRenderer()
      this.setStats()
      this.setControls()
      // this.generateGround()
      // this.setLight()
      this.loadGlb()
      this.render()
    },
    setCamera() {
      this.camera = new THREE.PerspectiveCamera(45, this.width/this.height, 1, 100)
      this.camera.position.set(5, 2, 8)
    },
    setScene() {
      this.scene = new THREE.Scene()
      this.scene.background = new THREE.Color(0xbfe3dd)
    },
    setRenderer() {
      this.renderer = new THREE.WebGLRenderer({antialias: true})
      this.renderer.setPixelRatio(window.devicePixelRatio)
      this.renderer.setSize(this.width, this.height)
      this.renderer.outputEncoding = THREE.sRGBEncoding;
      const container = document.getElementById('container')
      container.appendChild(this.renderer.domElement)
    },
    setStats() {
      const container = document.getElementById('container')
      this.stats = new Stats()
      container.appendChild(this.stats.dom)
    },
    setControls() {
      const controls = new OrbitControls(this.camera, this.renderer.domElement)
      controls.target.set(0, 0.5, 0)
      controls.update()
      controls.enablePan = false
      controls.enableDamping = true
    },
    loadGlb() {
      const dracoLoader = new DRACOLoader();
			dracoLoader.setDecoderPath( 'three/examples/js/libs/draco/gltf/' );
      const loader = new GLTFLoader()
      loader.setDRACOLoader( dracoLoader );
      loader.load('./LittlestTokyo.glb', (gltf) => {
        const model = gltf.scene
        this.scene.add(model)

      })
    },
    render() {
      requestAnimationFrame(this.render.bind(this))
      this.renderer.render(this.scene, this.camera)
      this.stats.update()
    }
  }
}
</script>

<style>
#container{
  position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
}
</style>