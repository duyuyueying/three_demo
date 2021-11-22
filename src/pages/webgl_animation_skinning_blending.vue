<template>
  <div id="container"></div>
</template>

<script>
import * as THREE from 'three'
export default {
    data() {
        return {
            camera: null,
            scene: null,
            renderer: null,
            width: window.innerWidth,
            height: window.innerHeight,
        }
    },
    mounted() {
        this.init()
    },
    methods: {
        init() {
            this.setCamera()
            this.setScene()
            this.setRenderer()
            this.render()
        },
        setCamera(){
            this.camera = new THREE.PerspectiveCamera(45, this.width / this.height, 1, 1000)
            this.camera.position.set(1,2,-3)
            this.camera.lookAt(0,1,0)
        },
        setScene() {
            this.scene = new THREE.Scene()
            this.scene.background = new THREE.Color(0xa0a0a0)
            this.scene.fog = new THREE.Fog(0xa0a0a0, 10, 50)
        },
        setRenderer() {
            this.renderer = new THREE.WebGLRenderer({antialias: true})
            this.renderer.setPixelRatio(window.devicePixelRatio)
            this.renderer.setSize(this.width, this.height)
            this.renderer.outputEncoding = THREE.sRGBEncoding;
            this.renderer.shadowMap.enabled = true;
            const container = document.getElementById('container')
            container.appendChild(this.renderer.domElement)
        },
        render() {
            this.renderer.render(this.scene, this.camera)
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