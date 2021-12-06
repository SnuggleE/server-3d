<template>
  <div class="home">
    <canvas id="canvas"></canvas>
    <div class="btns">
      <p>
        <button @click="addModel">添加模型</button>
      </p>
      <hr>
      <p>
        <input type="text" :value="modelUrl">
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
        <button>修改颜色</button>
        <button @click="moveToMesh">缩放</button>
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
      modelUrl: 'http://localhost:8080/'
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
        var light = new BABYLON.HemisphericLight('light1', new BABYLON.Vector3(0, 1, 0), scene)
        return scene

      }
      this.scene = createScene()
      engine.runRenderLoop(() => {
        this.scene.render()
      })
      window.addEventListener('resize', function() {
        console.log(engine)
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
        this.camera.setTarget(mesh)
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
    importModelFromFile() {
      console.log(this.modelUrl)
      BABYLON.SceneLoader.ImportMeshAsync('', this.modelUrl, "server.babylon", this.scene)
      .then(result => {
        console.log(result)
      });
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
    width: 800px;
    height: 800px;
    border: 1px solid #123;
  }
  .btns{
    padding: 10px;
  }
}
</style>
