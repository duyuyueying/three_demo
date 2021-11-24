<template>
  <div id="container"></div>
</template>

<script>
import * as THREE from 'three'
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader.js'
import * as SkeletonUtils from 'three/examples/jsm/utils/SkeletonUtils.js';
export default {
 data(){
    return {
      camera: null,
      scene: null,
      renderer: null,
      width: window.innerWidth,
      height: window.innerHeight,
      clock: null,
      stats: null,
      mixers: [],
      model: null, // 模型
      MODELS: [
				{ name: "Soldier" },
				{ name: "Parrot" },
			],
      UNITS: [
				{
					modelName: "Soldier", // Will use the 3D model from file models/gltf/Soldier.glb
					meshName: "vanguard_Mesh", // Name of the main mesh to animate
					position: { x: 0, y: 0, z: 0 }, // Where to put the unit in the scene
					scale: 1, // Scaling of the unit. 1.0 means: use original size, 0.1 means "10 times smaller", etc.
					animationName: "Idle" // Name of animation to run
				},
				{
					modelName: "Soldier",
					meshName: "vanguard_Mesh",
					position: { x: 3, y: 0, z: 0 },
					scale: 2,
					animationName: "Walk"
				},
				{
					modelName: "Soldier",
					meshName: "vanguard_Mesh",
					position: { x: 1, y: 0, z: 0 },
					scale: 1,
					animationName: "Run"
				},
				{
					modelName: "Parrot",
					meshName: "mesh_0",
					position: { x: - 4, y: 0, z: 0 },
					rotation: { x: 0, y: Math.PI, z: 0 },
					scale: 0.01,
					animationName: "parrot_A_"
				},
				{
					modelName: "Parrot",
					meshName: "mesh_0",
					position: { x: - 2, y: 0, z: 0 },
					rotation: { x: 0, y: Math.PI / 2, z: 0 },
					scale: 0.02,
					animationName: null
				},
			],
      numLoadedModels: 0
    }
  },
  mounted() {
      this.init()
  },
  methods: {
    init() {
      this.clock= new THREE.Clock()
      this.setCamera()
      this.setRenderer()
      this.setScene()
      this.setLight()
      this.setGround()
      for(let i = 0; i < this.MODELS.length; i++) {
        this.loadGLB(this.MODELS[i], () => {
          this.numLoadedModels++;
          if(this.numLoadedModels === this.MODELS.length) {
            this.setGLB()
          }
        })
      }
      this.render()
    },
    setCamera() {
      this.camera = new THREE.PerspectiveCamera(45, this.width/this.height, 1, 10000)
      this.camera.position.set(3,6, -10)
      this.camera.lookAt(0, 1, 0)
    }, 
    setRenderer() {
      this.renderer = new THREE.WebGLRenderer({antialias: true})
      this.renderer.setSize(this.width, this.height)
      this.renderer.setPixelRatio(window.devicePixelRatio)
      this.renderer.outputEncoding = THREE.sRGBEncoding;
      this.renderer.enabled = true;
      this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
      const container = document.getElementById('container')
      container.appendChild(this.renderer.domElement)
    },
    setScene() {
      this.scene = new THREE.Scene()
      this.scene.background = new THREE.Color(0xa0a0a0)
      this.scene.fog = new THREE.Fog(0xa0a0a0, 10, 22)
    },
    setLight() {
      const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444)
      hemiLight.position.set(0, 20, 0)
      this.scene.add(hemiLight)

      const dirLight = new THREE.DirectionalLight(0xffffff)
      dirLight.position.set(-3, 10, -10)
      dirLight.castShadow = true;
      dirLight.shadow.camera.top = 10;
      dirLight.shadow.camera.bottom = - 10;
      dirLight.shadow.camera.left = - 10;
      dirLight.shadow.camera.right = 10;
      dirLight.shadow.camera.near = 0.1;
      dirLight.shadow.camera.far = 40;
      this.scene.add(dirLight)
    },
    setGround() {
      const ground = new THREE.PlaneGeometry(40, 40)
      const material = new THREE.MeshPhongMaterial({
        color: 0x999999,
        depthWrite: false,
      })
      const groundMesh = new THREE.Mesh(ground, material)
      groundMesh.rotation.x = - Math.PI / 2;
			groundMesh.receiveShadow = true;
      this.scene.add(groundMesh)
    },
    loadGLB(model, onLoaded) {
      const loader = new GLTFLoader()
      loader.load('./'+model.name+'.glb', (gltf) => {
        const scene = gltf.scene
        model.scene = gltf.scene
        model.animations = gltf.animations
        scene.traverse((object) => {
          if(object.isMesh) {
            object.castShadow = true
          }
        })
        onLoaded()
      })
    },
    setGLB() {
      for(let i = 0; i < this.UNITS.length; i++) {
        const u = this.UNITS[i]
        const model = this.getModelByName(u.modelName)
        if(model) {
          const clonedScene = SkeletonUtils.clone( model.scene );
          if(clonedScene){
            const clonedMesh = clonedScene.getObjectByName( u.meshName );
            if(clonedMesh) {
              const mixer = this.setAnimation( clonedMesh, model.animations, u.animationName );
              this.mixers.push(mixer)
            }
            if(u.position) {
              clonedScene.position.set( u.position.x, u.position.y, u.position.z );
            }
            if(u.scale) {
              clonedScene.scale.set( u.scale, u.scale, u.scale );
            }
            if(u.rotation) {
              clonedScene.rotation.x = u.rotation.x;
              clonedScene.rotation.y = u.rotation.y;
              clonedScene.rotation.z = u.rotation.z;
            }
            this.scene.add(clonedScene)
          }
        }
      }
    },
    setAnimation(mesh, animations, animationName) {
      const mixer = new THREE.AnimationMixer(mesh)
      const clip = THREE.AnimationClip.findByName(animations, animationName)
      if(clip){
        const action = mixer.clipAction(clip)
        action.play()
      }
      return mixer

    },
    getModelByName( name ) {
				for ( let i = 0; i < this.MODELS.length; ++ i ) {
					if ( this.MODELS[ i ].name === name ) {
						return this.MODELS[ i ];
					}
				}
				return null;
			},
    render() {
      requestAnimationFrame(this.render)
      const mixerUpdateDelta = this.clock.getDelta()
      for(let i = 0; i < this.mixers.length; ++i) {
        this.mixers[i].update(mixerUpdateDelta)
      }
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