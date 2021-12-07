<template>
  <div class="home">
    <canvas id="canvas"></canvas>
    <div class="btns">
      <p>
        <button @click="addModel">添加模型</button>
      </p>
      <hr>
      <p>
        <input type="text" v-model="modelUrl">
        <button @click="importModelFromFile">导入模型</button>
      </p>
      <hr>
      <p>
        <select name="mode" v-model="selectMesh" id="">
          <option v-for="mesh in meshes" :key="mesh.id" :value="mesh.id">{{mesh.name}}</option>
        </select>
      </p>
      <p>
        <button @click="setVisibility">修改透明度</button>
        <button @click="setColor">修改颜色</button>
        <button @click="moveToMesh">缩放</button>
      </p>
      <hr>
      <p>
        <button @click="importDoor">导入门</button>
        <button @click="closeDoor">关门</button>

      </p>
      <hr>
    </div>
  </div>
</template>

<script>
import * as BABYLON from 'babylonjs'

export default {
  name: 'Home',
  data() {
    return {
      scene: null,
      camera: null,
      selectMesh: '',
      modelUrl: 'http://localhost:8082/'
    }
  },
  computed: {
    meshes() {
      if(this.scene) {
        return this.scene.meshes
      } else return {}
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      // 1、场景初始化
      var canvas = document.getElementById('canvas')
      var engine = new BABYLON.Engine(canvas, true, 
        {
          preserveDrawingBuffer: true,
          stencil: true
      })
      var createScene = () => {
        var scene = new BABYLON.Scene(engine)
        var camera = new BABYLON.ArcRotateCamera('camera',  Math.PI / 2, Math.PI/ 2, 5, new BABYLON.Vector3(0,0,0))
        camera.attachControl(canvas, true)
        this.camera = camera
        var light = new BABYLON.HemisphericLight('light1', new BABYLON.Vector3(0, 100, 0), scene)
        light.intensitySearch  = 1
        return scene

      }
      this.scene = createScene()
      engine.runRenderLoop(() => {
        this.scene.render()
      })
      window.addEventListener('resize', function() {
        engine.resize()
      })
    },
    addModel() {
      console.log(this.scene)
      var redBox = BABYLON.Mesh.CreateBox("red", 5, this.scene);
      var redMat = new BABYLON.StandardMaterial("ground", this.scene);
      redMat.diffuseColor = new BABYLON.Color3(0.4, 0.4, 0.4);
      redMat.specularColor = new BABYLON.Color3(0.4, 0.4, 0.4);
      redMat.emissiveColor = BABYLON.Color3.Red();
      redBox.material = redMat;
      redBox.position.x = 10;

      var sphere = BABYLON.Mesh.CreateSphere("sphere1", 16, 2, this.scene);

    // Move the sphere upward 1/2 its height
      sphere.position.y = 1;
    },
    moveToMesh() {
      if(this.selectMesh) {
        const mesh = this.scene.getMeshByID(this.selectMesh)
        this.camera.setTarget(mesh, {
          toBoundingCenter: true
        })
      } else {
        alert('请选择')
      }

    },
    setVisibility() {
      if(this.selectMesh) {
        const mesh = this.scene.getMeshByID(this.selectMesh)
        mesh.visibility = 0.2
      }
    },
    setColor() {
      if(this.selectMesh) {
        const mesh  = this.scene.getMeshByID(this.selectMesh)
        console.log(mesh)
        var redMat = new BABYLON.StandardMaterial("ground", this.scene);
        redMat.diffuseColor = new BABYLON.Color3(0.4, 0.4, 0.4);
        redMat.specularColor = new BABYLON.Color3(0.4, 0.4, 0.4);
        redMat.emissiveColor = BABYLON.Color3.Red();
        mesh.material = redMat
      }
    },
    importModelFromFile() {
      console.log(this.modelUrl)
      BABYLON.SceneLoader.Append(this.modelUrl, "server.babylon", this.scene, (result) => {
        console.log(result)
      })
    },
    importDoor() {
      BABYLON.SceneLoader.ImportMeshAsync('', this.modelUrl, 'door.babylon', this.scene)
      .then(res => {
        // const door = this.scene.getMeshByName('door')
        // door.scaling = new BABYLON.Vector3(10, 10, 10)
        // const doorAxis = this.scene.getMeshByName('Box001')
        // doorAxis.scaling = new BABYLON.Vector3(10, 10, 10)
      })
    },
    closeDoor() {
      const doorAxis = this.scene.getMeshByName('Box001')
      const door = this.scene.getMeshByName('door')
      const origin = doorAxis.getPivotPoint()
      const origin2 = origin.clone()
      origin2.y = 10
      if(door) {
        door.rotateAround(origin, origin2, Math.PI / 2, BABYLON.Space.LOCAL)
        this.scene.render()
      }
    }
  }

}
</script>

<style lang="less" scoped>
.home{
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: flex-start;
  #canvas{
    width: 1200px;
    height: 800px;
    border: 1px solid #123;
  }
  .btns{
    padding: 10px;
  }
}
</style>
