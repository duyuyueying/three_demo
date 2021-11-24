<template>
  <div id="container"></div>
</template>

<script>
import * as THREE from 'three'
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
      frustumSize: 600,
      aspect: window.innerWidth/window.innerHeight,
      cameraRig: null,
      mesh: null,
    }
  },
  mounted() {
      this.init()
  },
  methods: {
    init() {
      this.setScene()
      this.setCamera()
      this.setRenderer()
      this.genMesh()
      this.render()
    },
    setCamera() {
      this.camera = new THREE.PerspectiveCamera(50, 0.5 * this.aspect, 1, 10000)
      this.camera.position.z = 2500

      this.cameraPerspective = new THREE.PerspectiveCamera( 50, 0.5 * this.aspect, 150, 1000 );
      const cameraPerspectiveHelper = new THREE.CameraHelper(this.cameraPerspective)
      this.scene.add(cameraPerspectiveHelper)


      const cameraOrtho = new THREE.OrthographicCamera( 0.5 * this.frustumSize * this.aspect / - 2, 0.5 * this.frustumSize * this.aspect / 2, this.frustumSize / 2, this.frustumSize / - 2, 150, 1000 );

			const cameraOrthoHelper = new THREE.CameraHelper( cameraOrtho );
			this.scene.add( cameraOrthoHelper );
      cameraOrtho.rotation.y = Math.PI;
      this.cameraPerspective.rotation.y = Math.PI

      this.cameraRig = new THREE.Group()
      this.cameraRig.add(this.cameraPerspective)
      this.cameraRig.add(cameraOrtho)

      this.scene.add(this.cameraRig)
      
    }, 
    setRenderer() {
      this.renderer = new THREE.WebGLRenderer({antialias: true})
      this.renderer.setPixelRatio(window.devicePixelRatio)
      this.renderer.setSize(this.width, this.height)
      const container = document.getElementById('container')
      container.appendChild(this.renderer.domElement)
    },
    setScene() {
      this.scene = new THREE.Scene()
    },
    genMesh() {
      const sphere = new THREE.SphereGeometry(100, 16, 8)
      const material = new THREE.MeshBasicMaterial({color: 0xffffff, wireframe: true})
      this.mesh = new THREE.Mesh(sphere, material)
      this.scene.add(this.mesh)

      const mesh2 = new THREE.Mesh(new THREE.SphereGeometry(50, 16, 8), new THREE.MeshBasicMaterial({color: 0x00ff00, wireframe: true}))
      this.scene.add(mesh2)
      mesh2.position.y = 150

      const mesh3 = new THREE.Mesh(
        new THREE.SphereGeometry( 5, 16, 8 ),
        new THREE.MeshBasicMaterial( { color: 0x0000ff, wireframe: true } )
      );
      mesh3.position.z = 150;
      this.cameraRig.add( mesh3 );

      const geometry = new THREE.BufferGeometry()
      const vertices = []
      for( let i = 0; i < 100000; i++) {
        vertices.push(THREE.MathUtils.randFloatSpread( 2000 ))
        vertices.push(THREE.MathUtils.randFloatSpread( 2000 ))
        vertices.push(THREE.MathUtils.randFloatSpread( 2000 ))
      }
      geometry.setAttribute('position', new THREE.Float32BufferAttribute(vertices, 3))
      const particles = new THREE.Points(geometry, new THREE.PointsMaterial({color: 0x888888}))
      this.scene.add(particles)
    },
    render() {
      this.renderer.setViewport(0,0,this.width/2, this.height)
      this.renderer.render(this.scene, this.camera)
      this.renderer.setViewport(this.width/2, 0, this.width/2, this.height)
      this.renderer.render(this.scene, this.cameraPerspective)
    }
  }
}
</script>

<style>

</style>