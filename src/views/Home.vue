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
          <option v-for="mesh in meshes" :key="mesh.id" :value="mesh.name">{{mesh.name}}</option>
        </select>
      </p>
      <p>
        <button @click="setVisibility">修改透明度</button>
        <button @click="setColor">修改颜色</button>
        <button @click="moveToMesh">缩放</button>
      </p>
      <hr>
      <p>
        <button @click="toggleDoor()">开/关门</button>
      </p>
      <hr>
    </div>
  </div>
</template>

<script>
import * as BABYLON from 'babylonjs'
import * as GUI from 'babylonjs-gui'

export default {
  name: 'Home',
  data() {
    return {
      scene: null,
      camera: null,
      selectMesh: '',
      modelUrl: 'http://localhost:8081/',
      doorStatus: false
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
      this.importModelFromFile()
    },
    addModel() {
      var redBox = BABYLON.Mesh.CreateBox("red", 5, this.scene);
      var redMat = new BABYLON.StandardMaterial("ground", this.scene);
      redMat.diffuseColor = new BABYLON.Color3(0.4, 0.4, 0.4);
      redMat.specularColor = new BABYLON.Color3(0.4, 0.4, 0.4);
      redMat.emissiveColor = BABYLON.Color3.Red();
      redBox.material = redMat;
      redBox.position.x = 10;

      var sphere = BABYLON.Mesh.CreateSphere("sphere1", 16, 2, this.scene);

      sphere.position.y = 1;
    },
    moveToMesh() {
      if(this.selectMesh) {
        const mesh = this.scene.getMeshByName(this.selectMesh)
        this.camera.setTarget(mesh, {
          toBoundingCenter: true
        })
      } else {
        alert('请选择')
      }

    },
    setVisibility() {
      console.log(this.selectMesh)
      if(this.selectMesh) {
        const mesh = this.scene.getMeshByName(this.selectMesh)
        console.log(mesh)
        mesh.visibility = 0.2
      }
    },
    setColor() {
      if(this.selectMesh) {
        const mesh  = this.scene.getMeshByName(this.selectMesh)
        console.log(mesh)
        var redMat = new BABYLON.StandardMaterial("ground", this.scene);
        redMat.diffuseColor = new BABYLON.Color3(0.4, 0.4, 0.4);
        redMat.specularColor = new BABYLON.Color3(0.4, 0.4, 0.4);
        redMat.emissiveColor = BABYLON.Color3.Red();
        mesh.material = redMat
      }
    },
    importModelFromFile() {
      BABYLON.SceneLoader.ImportMeshAsync('', this.modelUrl, "server-room.babylon", this.scene)
      .then(res => {
        console.log(this.scene.meshes)
        const wall = this.scene.getMeshByName('Line001')
        this.camera.setTarget(wall)
        this.showBrand()
      })
    },
    showBrand() {
      const meshes = this.scene.meshes
      meshes.forEach(mesh => {
        if(mesh.name && mesh.name.indexOf('cabinet') > -1) {
          const index = mesh.name.replace('cabinet', '')

          var plane = BABYLON.Mesh.CreatePlane("plane" + index, 1);
          plane.parent = mesh;
          plane.position.y = 2.5;

          plane.billboardMode = BABYLON.Mesh.BILLBOARDMODE_ALL;
          var advancedTexture = GUI.AdvancedDynamicTexture.CreateForMesh(plane);
          var button1 = GUI.Button.CreateSimpleButton("but" + index, mesh.name);
          button1.width = 1;
          button1.height = 0.4;
          button1.color = "white";
          button1.fontSize = 90;
          button1.background = "green";
          button1.metadata = {
            origin: mesh.name
          }
          button1.onPointerUpObservable.add(function(e, state) {
            console.log(state.target)
              // alert("you did it!");
          });
          advancedTexture.addControl(button1);


        }
      })
    },
    toggleDoor() {
      const doorAxis = this.scene.getMeshByName('doorAxis')
      const door = this.scene.getMeshByName('door')
      const origin = doorAxis.getAbsolutePivotPoint()
      const origin2 = new BABYLON.Vector3(0, 1, 0)
      const angle = this.doorStatus ? Math.PI / 2 : -Math.PI / 2
      if(door) {
        door.rotateAround(origin, origin2, angle)
        this.scene.render()
        this.doorStatus = !this.doorStatus
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
  }
  .btns{
    padding: 10px;
  }
}
</style>
