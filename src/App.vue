<template>
  <Renderer ref="renderer" antialias :pointer="{ onMove: onPointerMove }" resize :orbit-ctrl="{ enableDamping: true }" @click="onClick">
    <Camera :position="{ x: 0, y: 20, z: 0 }" />
    <Scene ref="scene" :background="cubeTexture">
      <!-- <AmbientLight color="#ffffff" /> -->

      <!-- <PointLight :intensity="0.5" color="#ffffff" :position="{ x: -50, y: 30, z: -50}" /> -->

      <Group :rotation="{ x: -Math.PI / 2 }">
        <!-- <SpotLight
          color="#ffffff"
          :intensity="1"
          :position="{ z: 15 }"
          :angle="Math.PI / 5"
          :penumbra="1"
        /> -->
        <LiquidPlane ref="liquid"
          :width="WIDTH" :height="HEIGHT"
          :material-props="materialProps"
        />

        <PlaneC
          :position="{ z: -2 }"
          :width="WIDTH" :height="HEIGHT"
        >
          <BasicMaterial>
            <Texture
              src="/water/assets/tiles/Tiles107_1K_Color.jpg"
              :props="{ repeat: { x: 1.5, y: 1.5 }, wrapS: RepeatWrapping, wrapT: RepeatWrapping }"
            />
          </BasicMaterial>
        </PlaneC>

        <!-- <Box :width="WIDTH" :height="HEIGHT" :depth="0.2" :position="{ x: 0, y: 0, z: -2.4 }">
          <BasicMaterial>
            <Texture
              src="/water/assets/tiles/Tiles107_1K_Color.jpg"
              :props="{ repeat: { x: 1, y: 1 }, wrapS: RepeatWrapping, wrapT: RepeatWrapping }"
            />
          </BasicMaterial>
        </Box> -->

        <!-- <Box :width="WIDTH" :height="0.2" :depth="3" :position="{ x: 0, y: HEIGHT / 2 + 0.1, z: -1 }">
          <BasicMaterial>
            <Texture
              src="/water/assets/wood/Wood048_1K_Color.jpg"
              :props="{ repeat: { x: 2, y: 0.5 }, wrapS: RepeatWrapping, wrapT: RepeatWrapping }"
            />
          </BasicMaterial>
        </Box>
        <Box :width="WIDTH" :height="0.2" :depth="3" :position="{ x: 0, y: -HEIGHT / 2 - 0.1, z: -1 }">
          <BasicMaterial>
            <Texture
              src="/water/assets/wood/Wood048_1K_Color.jpg"
              :props="{ repeat: { x: 2, y: 0.5 }, wrapS: RepeatWrapping, wrapT: RepeatWrapping }"
            />
          </BasicMaterial>
        </Box>

        <Box :width="0.2" :height="HEIGHT + 0.4" :depth="3" :position="{ x: WIDTH / 2 + 0.1, y: 0, z: -1 }">
          <BasicMaterial>
            <Texture
              src="/water/assets/wood/Wood048_1K_Color.jpg"
              :props="{ repeat: { x: 0.5, y: 2 }, wrapS: RepeatWrapping, wrapT: RepeatWrapping }"
            />
          </BasicMaterial>
        </Box>
        <Box :width="0.2" :height="HEIGHT + 0.4" :depth="3" :position="{ x: -WIDTH / 2 - 0.1, y: 0, z: -1 }">
          <BasicMaterial>
            <Texture
              src="/water/assets/wood/Wood048_1K_Color.jpg"
              :props="{ repeat: { x: 0.5, y: 2 }, wrapS: RepeatWrapping, wrapT: RepeatWrapping }"
            />
          </BasicMaterial>
        </Box> -->

      </Group>

    </Scene>
  </Renderer>
</template>

<script>
import { CubeTextureLoader, Plane, Raycaster, RepeatWrapping, Vector3 } from 'three'
import { Pane } from 'tweakpane'

import {
  BasicMaterial,
  Box,
  Camera,
  Group,
  Plane as PlaneC,
  Renderer,
  Scene,
  Texture
} from 'troisjs'

import LiquidPlane from '@troisjs/components/src/liquid/LiquidPlane.js'

export default {
  components: {
    BasicMaterial,
    Box,
    Camera,
    Group,
    LiquidPlane,
    PlaneC,
    Renderer,
    Scene,
    Texture
  },
  setup() {
    const loader = new CubeTextureLoader()
    // loader.setPath('https://threejs.org/examples/textures/cube/skyboxsun25deg/')
    // const cubeTexture = loader.load([
    //   'px.jpg', 'nx.jpg',
    //   'py.jpg', 'ny.jpg',p
    //   'pz.jpg', 'nz.jpg'
    // ])

    loader.setPath('https://threejs.org/examples/textures/cube/Park2/')
    const cubeTexture = loader.load([
      'posx.jpg', 'negx.jpg',
      'posy.jpg', 'negy.jpg',
      'posz.jpg', 'negz.jpg'
    ])

    return {
      WIDTH: 20,
      HEIGHT: 20,
      RepeatWrapping: RepeatWrapping,
      cubeTexture
    }
  },
  data() {
    return {
      isPlaying: true,
      materialProps: {
        color: 0xffffff,
        metalness: 0.2,
        roughness: 0,
        thickness: 4,
        transmission: 1,
        displacementScale: 20,
        envMap: this.cubeTexture,
        envMapIntensity: 1
      },
      textureProps: {
        repeat: { x: 2, y: 2 },
        wrapS: RepeatWrapping,
        wrapT: RepeatWrapping
      }
    }
  },
  mounted() {
    this.pointer = this.$refs.renderer.three.pointer
    this.liquidEffect = this.$refs.liquid.liquidEffect

    // custom raycaster
    this.raycaster = new Raycaster()
    this.pointerPlane = new Plane(new Vector3(0, 1, 0), 0)
    this.pointerV3 = new Vector3()

    document.body.addEventListener('keyup', this.onKeyup)

    this.pane = new Pane()
    this.pane.addInput(this.materialProps, 'metalness', { step: 0.05, min: 0, max: 1 })
    this.pane.addInput(this.materialProps, 'roughness', { step: 0.05, min: 0, max: 1 })
    this.pane.addInput(this.materialProps, 'thickness', { step: 0.05, min: 0, max: 10 })
    this.pane.addInput(this.materialProps, 'transmission', { step: 0.05, min: 0, max: 1 })
    this.pane.addInput(this.materialProps, 'displacementScale', { step: 0.5, min: 0.5, max: 50 })
    this.pane.addInput(this.materialProps, 'envMapIntensity', { step: 0.05, min: 0, max: 1 })
    this.pane.addInput(this.materialProps, 'color', { view: 'color' })

    this.pane.addButton({ title: 'Pause/Play' }).on('click', () => { this.togglePlay() })
  },
  unmounted() {
    this.pane.dispose()
  },
  methods: {
    togglePlay() {
      this.isPlaying = !this.isPlaying
      this.$refs.liquid.togglePlay()
    },
    onKeyup(e) {
      if (e.keyCode === 32) {
        this.togglePlay()
      } else if (e.keyCode === 82 && this.isPlaying) {
        for (let i = 0; i < 20; i++) {
          this.liquidEffect.addDrop(Math.random() * 2 - 1, Math.random() * 2 - 1, 0.01, 0.025)
        }
      }
    },
    onClick() {
      if (this.isPlaying) {
        const x = 2 * this.pointerV3.x / this.WIDTH
        const y = 2 * -this.pointerV3.z / this.HEIGHT
        this.liquidEffect.addDrop(x, y, 0.025, 0.05)
      }
    },
    onPointerMove() {
      if (this.isPlaying) {
        this.raycaster.setFromCamera(this.pointer.positionN, this.$refs.renderer.three.camera)
        this.raycaster.ray.intersectPlane(this.pointerPlane, this.pointerV3)
        const x = 2 * this.pointerV3.x / this.WIDTH
        const y = 2 * -this.pointerV3.z / this.HEIGHT
        this.liquidEffect.addDrop(x, y, 0.025, 0.002)
      }
    }
  }
}
</script>

<style>
body, html, #app {
  margin: 0;
  height: 100%;
}
canvas {
  display: block;
}
</style>
