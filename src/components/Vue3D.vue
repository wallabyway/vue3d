<template>
  <div class="vue3d">
    <h1>{{ msg }}</h1>
    <h3>{{ about }}<br/>
      Source <a v-bind:href="linkto"> vue3d </a> Contact <a v-bind:href="mailto">GreenPDX</a></h3>
    <div id="three3d" class="three3d" ref="three3d">
      <h2>{{ selectIt }}</h2>
      <div id="container">
        <div ref="overlay" id="overlay"
          @click="clickIt($event)" :size="size">
          <worm-hole :zoom1="zoom1" :zoom2="zoom2"></worm-hole>
        </div>
        <v3d-renderer id="renderer" ref="renderer" :size="size" :orbit="controls">
          <v3d-scene ref="scene">
            <v3d-orbit-controls ref="controls">
              <v3d-camera ref="camera0" :position="camPos"></v3d-camera>
            </v3d-orbit-controls>
            <v3d-light color="#ffffff"></v3d-light>
            <v3d-group>
              <v3d-grid :nodes="nodes"
                v-on:click.capture.stop="clickIt($event)">
              </v3d-grid>
            </v3d-group>
          </v3d-scene>
        </v3d-renderer>
        <div v-show="showInfo" class="infopop" ref="infopop">
          <div v-html="objInfo"></div>
        </div>
      </div>
    </div>
    <div class="right">
      <h3>Template Code</h3>
      <button v-on:click="loadClick($event)">Load</button>
      <button v-on:click="clrClick($event)">Clear</button>
      <button v-on:click="dumpClick($event)">Dump</button>
    </div>
  </div>
</template>

<script>
// import Vue from 'vue'
import { mapGetters, mapActions } from 'vuex'
import * as THREE from 'three'
import axios from 'axios'

// import Object3D from '@/components/Object3D'
import Renderer from './vue3d/Renderer'
import Scene from './vue3d/Scene'
import Camera from './vue3d/Camera'
import Light from './vue3d/Light'
import Mesh from './vue3d/Mesh'
import Geometry from './vue3d/Geometry'
import Material from './vue3d/Material'
import OrbitControls from './vue3d/OrbitControls'
import Group from './vue3d/Group'
import Grid from './vue3d/Grid'
import WormHole from './vue3d/WormHole'

import TestData from '@/assets/TestData'

export default {
  name: 'vue3d',

  components: {
    'v3d-renderer': Renderer,
    'v3d-scene': Scene,
    'v3d-camera': Camera,
    'v3d-light': Light,
    'v3d-mesh': Mesh,
    'v3d-geometry': Geometry,
    'v3d-material': Material,
    'v3d-orbit-controls': OrbitControls,
    'v3d-group': Group,
    'v3d-grid': Grid,
    'worm-hole': WormHole
  },

  data () {
    return {
      msg: 'Vue and Three JS test project',
      version3d: '',
      objInfo: '',
      threeSize: {w: '800px', h: '800px'},
      selectIt: 'Double click to select cube, mouse down to rotate, wheel to zoom',
      about: '3D framework for vue ',
      linkto: 'https://github.com/greenpdx/vue3d',
      mailto: 'mailto:savages@taxnvote.org?subject=vue3d%20Demo%20',
      grppos: {'x': 0, 'y': 10, 'z': 0},
      beacat: new Set(),
      year: '2016',
      data: {},
      nodes: [],
      controls: null,
      size: {x: 800, y: 800},
      zoom1: 1,
      zoom2: 1,
      camPos: {x: 50, y: 150, z: 100}
    }
  },

  beforeCreate () {
    this.td = new TestData()
    this.data = this.td.genData().top
    console.log(this.data)
  },

  created () {
    this.td = new TestData()
    this.data = this.td.genData().top
    console.log(this.data)

    this.beacat.add('Discretionary')
    this.version3d = THREE.REVISION
    let rand = Math.floor(Math.random() * (Number.MAX_SAFE_INTEGER - 1))
    this.mailto = this.mailto + document.referrer + '&body=Please Keep; ' + rand
    this.setThree3d(this)
    this.nodes = this.data.children[0].children
    console.log(this.nodes)
  },

  mounted () {
    let ele = this.$refs.infopop
    this.controls = this.$refs.controls
    ele.style.top = this.mySize.top + 'px'
    ele.style.left = this.mySize.left + 'px'
  },

  computed: {
    ...mapGetters('three3d', {
      renderer: 'renderer',
      scene: 'scene',
      cameraV: 'camera'
    }),
    ...mapGetters({
      hoverObj: 'hoverObj',
      selectObj: 'selectObj'
    }),
    ...mapGetters('nodes', {
      dumpNodes: 'dumpNodes',
      getNodes: 'nodes'
    }),
    showInfo: function () {
      if (this.hoverObj !== null) {
        let obj = this.hoverObj
        let str = ''
        let node = obj.node
        str += obj.id3d + ' ' + node.value + '<br />'
        str += node.name + '<br />'
        str += Math.round(obj.loc.x * 100) / 100 +
          ', ' + Math.round(obj.loc.y * 100) / 100 +
          ', ' + Math.round(obj.loc.z * 100) / 100
        this.objInfo = str
        return true
      } else {
        this.objInfo = ''
        return false
      }
    },
    mySize: function () {
      let ren = this.$refs.renderer.$el
      return {
        top: ren.offsetTop,
        left: ren.offsetLeft,
        width: ren.offsetWidth,
        height: ren.offsetHeight,
        bottom: ren.offsetTop + ren.offsetHeight,
        right: ren.offsetLeft + ren.offsetWidth
      }
    }
  },
  methods: {
    ...mapActions('three3d', [
      'setThree3d'
    ]),
    ...mapActions('nodes', [
      'setNodes',
      'clearNodes',
      'getNodeById'
    ]),
    filterNodes (itm, idx) {
      if (!this.beacat.has(itm.beacat)) {
        return null
      }
      if (itm[this.year] === 0) {
        return null
      }

      return {idx: idx, id: itm._id}
    },
    loadClick (evt) {   // try bezier  gre/bezier-easing
      let camera = this.$refs.camera0
      let sel = this.selectObj
      let old = camera.curObj.position
      console.log('CAM', old)
      if (sel !== null) {
        console.log(sel.loc)    // just change

/*        let dif.x = (old.x - sel.loc.x) / 100
        let dif.y = (old.y - sel.loc.y) / 100
        let dif.z = (old.z -sel.loc.z) / 100
        let cnt =
        let id = setInterval(function () {
          if (cnt-- < 0) {

          }
          camera.curObj.position.set(sel.loc.x, sel.loc.y + 50, sel.loc.z)
          camera.curObj.lookAt(new THREE.Vector3(sel.loc.x, sel.loc.y, sel.loc.z))
        }, 10) */
      }
    },
    clrClick (evt) {
      this.clearNodes()
    },
    clickIt (evt) {
      console.log(evt)
    },
    zoomIn (node) {

    },
    dumpClick (evt) {
      let nodes = this.getNodes
      nodes.forEach((itm, idx) => {
        let val = this.getNodeById(itm._id)
        val.then(node => console.log(itm._id, node._id))
      })
      let itm = this.getNodeById(this.tstId)
      itm.then(node => console.log(this.tstId, node._id))
    },
    getData () {
      let self = this
      axios.get('http://10.0.42.81:8181/docs/local/budget/full')
        .then(response => {
          let rslt = response.data
          let data = rslt.data
          self.setNodes(data)
        })
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.vue3d {
    display: inline;
}
.three3d {
  float: left;
  width: 48%;
  height: 800px;
}
.infopop {
  border: 2 solid black;
  width: 30em;
  height: 100px;
  position: absolute;
  display: inline-block;
  top: 0px;
  left: 0px;
  background-color: yellow;
  z-index: 10;
}
.right {
  float: right;
  width: 45%;
  text-align: left;
  font-size: 1.2em;
}
#container {
  position: relative;
}
#renderer {
  position: absolute;
  top: 0;
  left: 0;
  margin-top: 40;
}
#overlay {
  position: absolute;
  z-index: 10;
  top: -20;
  left: 0;
  width: 800px;
  height: 800px;
  pointer-events: none;
}
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
