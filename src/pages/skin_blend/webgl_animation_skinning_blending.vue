<template>
  <div id="container"></div>
</template>

<script>
import * as THREE from 'three'
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader.js'
import Stats from 'three/examples/jsm/libs/stats.module.js';
import { GUI } from 'three/examples/jsm/libs/dat.gui.module.js';
export default {
    data() {
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
            this.generateGround()
            this.setLight()
            this.loadGlb()
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
        setStats() {
            const container = document.getElementById('container')
            this.stats = new Stats();
			container.appendChild( this.stats.dom );
        },
        setLight() {
            const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444)
            hemiLight.position.set(0, 20, 0)
            var helper = new THREE.HemisphereLightHelper( hemiLight, 5 );
            this.scene.add( helper );
            const dirLight = new THREE.DirectionalLight(0xffffff)
            dirLight.position.set(-3, 10, -10)
            dirLight.castShadow = true;
            dirLight.shadow.camera.top = 2;
            dirLight.shadow.camera.bottom = - 2;
            dirLight.shadow.camera.left = - 2;
            dirLight.shadow.camera.right = 2;
            dirLight.shadow.camera.near = 0.1;
            dirLight.shadow.camera.far = 40;
            this.scene.add(hemiLight)
            this.scene.add(dirLight)
        },
        generateGround() {
            const plane = new THREE.PlaneGeometry(100, 100);
            const material = new THREE.MeshPhongMaterial({color: 0x999999, depthWrite: false})
            const mesh = new THREE.Mesh(plane, material)
            mesh.rotation.x = -Math.PI / 2
            mesh.receiveShadow = true
            this.scene.add(mesh)
        },
        loadGlb() {
            const loader = new GLTFLoader()
            loader.load('./Soldier.glb', (gltf) => {
                this.model = gltf.scene
                this.scene.add(this.model)
                this.model.traverse((object) => {
                    if(object.isMesh) object.castShadow = true
                })
                // 显示骨架
                this.skeleton = new THREE.SkeletonHelper(this.model)
                this.skeleton.visible = false;
                this.scene.add(this.skeleton)

                const animations = gltf.animations
                this.mixer = new THREE.AnimationMixer(this.model)
                this.idleAction = this.mixer.clipAction(animations[0])
                this.walkAction = this.mixer.clipAction(animations[3])
                this.runAction = this.mixer.clipAction(animations[1])
                this.actions = [this.idleAction, this.walkAction, this.runAction]
                this.createPanel()
                this.activateAllActions()
                this.render()
            })
        },
        activateAllActions() {
            this.setWight(this.idleAction, this.setting['modify idle weight'])
            this.setWight(this.walkAction, this.setting['modify walk weight'])
            this.setWight(this.runAction, this.setting['modify run weight'])
            this.actions.forEach((action) => {
                action.play()
            })
        },
        setWight(action, weight) {
            action.enabled = true
            action.setEffectiveTimeScale(1)
            action.setEffectiveWeight(weight)
        },
        createPanel() {
            const panel = new GUI({width: 310})
            const folder1 = panel.addFolder( 'Visibility' );
            // const folder6 = panel.addFolder( 'General Speed' );
            folder1.add(this.setting, 'show model').onChange(this.showModel)
            folder1.add(this.setting, 'show skeleton').onChange(this.showSkeleton)
            folder1.open()
            const folder2 = panel.addFolder( 'Activation/Deactivation' );
            folder2.add(this.setting, 'deactivate all')
            folder2.add(this.setting, 'activate all')
            folder2.open()
            const folder3 = panel.addFolder( 'Pausing/Stepping' );
            folder3.add(this.setting, 'pause/continue')
            folder3.add(this.setting, 'make single step')
            folder3.add(this.setting, 'modify step size', 0.01, 0.1, 0.001)
            folder3.open()
            const folder4 = panel.addFolder( 'Crossfading' );
            this.crossFadeControls.push(folder4.add(this.setting, 'from walk to idle'))
            this.crossFadeControls.push(folder4.add(this.setting, 'from idle to walk'))
            this.crossFadeControls.push(folder4.add(this.setting, 'from walk to run'))
            this.crossFadeControls.push(folder4.add(this.setting, 'from run to walk'))
            folder4.add( this.setting, 'use default duration' );
			folder4.add( this.setting, 'set custom duration', 0, 10, 0.01 );
            folder4.open()
            this.crossFadeControls.forEach((control)=>{
                control.classList1 = control.domElement.parentElement.parentElement.classList
                control.classList2 = control.domElement.previousElementSibling.classList
                control.setDisabled = function() {
                    control.classList1.add('no-pointer-events')
                    control.classList2.add('control-disabled')
                }
                control.setEnabled = function () {
                    control.classList1.remove( 'no-pointer-events' );
                    control.classList2.remove( 'control-disabled' );
                };
            })
            const folder5 = panel.addFolder( 'Blend Weights' );
            folder5.add(this.setting, 'modify idle weight', 0.0, 1.0, 0.01).onChange((weight) => this.setWight(this.idleAction, weight))
            folder5.add(this.setting, 'modify run weight', 0.0, 1.0, 0.01).onChange((weight) => this.setWight(this.runAction, weight))
            folder5.add(this.setting, 'modify walk weight', 0.0, 1.0, 0.01).onChange((weight) => this.setWight(this.walkAction, weight))
            folder5.open()
            const folder6 = panel.addFolder( 'General Speed' );
            folder6.add(this.setting, 'modify time scale', 0.0, 1.0, 0.01).onChange(this.modifyTimeScale)
            folder6.open()

        },
        showModel(visibility) {
            this.model.visible = visibility
        },
        showSkeleton(visibility) {
            this.skeleton.visible = visibility
        },
        deactivateAllActions() {
            this.actions.forEach((action) => {
                action.stop()
            })
        },
        pauseContinue() {
            if(this.singleStepMode) {
                this.singleStepMode = false
                this.unPauseAllActions()
            } else {
                if(this.idleAction.paused) {
                    this.unPauseAllActions()
                } else {
                    this.pauseAllActions()
                }
            }
        },
        unPauseAllActions() {
            this.actions.forEach((action) => {
                action.paused = false
            })
        },
        pauseAllActions() {
            this.actions.forEach((action) => {
                action.paused = true
            })
        },
        toSingleStepMode() {
            this.unPauseAllActions()
            this.singleStepMode = true
            this.sizeOfNextStep = this.setting['modify step size']
        },
        prepareCrossFade(startAction, endAction, defaultDuration) {
            const duration = this.setCrossFadeDuration(defaultDuration)
            this.singleStepMode = false
            this.unPauseAllActions()
            if(startAction === this.idleAction) {
                this.executeCrossFade(startAction, endAction, duration)
            } else {
                this.synchronizeCrossFade( startAction, endAction, duration );
            }
        },
        setCrossFadeDuration(defaultDuration) {
            if(this.setting['use default duration']) {
                return defaultDuration
            } else {
                return this.setting('set custom duration')
            }
        },
        executeCrossFade(startAction, endAction, duration) {
            this.setWight(endAction, 1)
            endAction.time = 0
            startAction.crossFadeTo(endAction, duration, true)
        },
        synchronizeCrossFade(startAction, endAction, duration) {
            const onLoopFinished = (event) =>  {
                if(event.action === startAction ){
                    this.mixer.removeEventListener( 'loop', onLoopFinished);
                    this.executeCrossFade(startAction, endAction, duration)
                }
            }
            this.mixer.addEventListener('loop', onLoopFinished)
        },
        updateCrossFadeControls() {
            this.crossFadeControls.forEach((control) => {
                control.setDisabled()
            });
            const idleWeight = this.idleAction.getEffectiveWeight();
			const walkWeight = this.walkAction.getEffectiveWeight();
			const runWeight = this.runAction.getEffectiveWeight();
            if(idleWeight == 1 && walkWeight === 0 && runWeight === 0) {
                this.crossFadeControls[1].setEnabled()
            }
            if ( idleWeight === 0 && walkWeight === 1 && runWeight === 0 ) {
                this.crossFadeControls[ 0 ].setEnabled();
                this.crossFadeControls[ 2 ].setEnabled();
            }
            if ( idleWeight === 0 && walkWeight === 0 && runWeight === 1 ) {
                this.crossFadeControls[ 3 ].setEnabled();
            }
        },
        modifyTimeScale(speed) {
            this.mixer.timeScale = speed
        },
        render() {
            requestAnimationFrame(this.render.bind(this))
            this.updateCrossFadeControls()
            let mixerUpdateDelta = this.clock.getDelta()
            if(this.singleStepMode) {
                mixerUpdateDelta = this.sizeOfNextStep
                this.sizeOfNextStep = 0
            }
            this.mixer.update(mixerUpdateDelta)
            this.stats.update()
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
.no-pointer-events {
				pointer-events: none;
			}
        .control-disabled {
            color: #888;
            text-decoration: line-through;
        }
</style>