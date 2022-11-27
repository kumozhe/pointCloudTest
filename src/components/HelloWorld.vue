<template>
  <div
    id="mainView"
    @click="onMouseClick"
    @mousemove="onMouseMove"
    @mousedown="onMouseDown"
    @mouseup="onMouseUp"
  ></div>
</template>
<script type="module">
import * as THREE from "three";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";
export default {
  name: "helloWorld",
  data() {
    return {
      scene: null,
      renderer: null,
      camera: null,
      controls: null,
      raycaster: null,

      //   linesSet: [],
      pickedObject: null, // 鼠标拾取到的对象
    };
  },
  mounted() {
    this.init();
    this.render();
    // this.animate();
  },
  methods: {
    init() {
      let view = document.getElementById("mainView");
      const pixelRatio = window.devicePixelRatio;
      let width = (view.clientWidth * pixelRatio) | 0;
      let height = (view.clientHeight * pixelRatio) | 0;
      const renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setSize(width, height);
      view.appendChild(renderer.domElement);

      const camera = new THREE.OrthographicCamera(
        (-1 * width) / 30,
        (1 * width) / 30,
        (1 * height) / 30,
        (-1 * height) / 30,
        -10,
        1000
      );
      camera.position.z = 50;

      // controls
      const controls = new OrbitControls(camera, renderer.domElement);
      //   controls.listenToKeyEvents(window); // optional
      controls.update();
      controls.addEventListener("change", this.render);

      //controls.addEventListener( 'change', render ); // call this only in static scenes (i.e., if there is no animation loop)

      //   controls.enableDamping = true; // an animation loop is required when either damping or auto-rotation are enabled
      //   controls.dampingFactor = 0.05;

      //   controls.screenSpacePanning = false;

      //   controls.minDistance = 100;
      //   controls.maxDistance = 500;

      //   controls.maxPolarAngle = Math.PI / 2;

      const scene = new THREE.Scene();

      const parentTransform = new THREE.Object3D();
      parentTransform.position.x = 0;
      parentTransform.position.y = 0;
      parentTransform.position.z = 0;
      this.parentTransform = parentTransform;
      this.createLines();
      scene.add(this.parentTransform);

      this.scene = scene;
      this.renderer = renderer;
      this.camera = camera;
      this.controls = controls;

      const color = 0xffffff;
      const intensity = 1;
      const light = new THREE.DirectionalLight(color, intensity);
      light.position.set(-1, 2, 4);
      scene.add(light);
    },
    onMouseUp(event) {
      console.log("start mouse up", event);
    },
    onMouseDown(event) {
      console.log("start mouse down");
      this.raycaster = new THREE.Raycaster();
      this.raycaster.params.Line.threshold = 3;
      this.pickedObjectSavedColor = null;

      // 发出射线
      let mouse = this.getMousePos(event);
      this.raycaster.setFromCamera(mouse, this.camera);
      // 获取与射线相交的对象
      const intersectedObjects = this.raycaster.intersectObjects(
        this.parentTransform.children,
        true
      );

      if (intersectedObjects.length > 0) {
        // 找到第一个对象，它是离鼠标最近的对象
        this.pickedObject = intersectedObjects[0].object;
        // 保存它的颜色
        this.pickedObjectSavedColor = this.pickedObject.material.color.getHex();
        // 设置它的发光为 黄色/红色闪烁
        this.pickedObject.material.color.setHex(0xffff00);
      } else {
        this.pickedObject = null;
      }
      this.render();
    },
    onMouseMove() {
      //   if (this.pickedObject) {
      //     this.controls.enableRotate = false;
      //     this.controls.enablePan = false;
      //     console.log(this.pickedObject);
      //   }
    },
    getMousePos(event) {
      const mouse = new THREE.Vector2();
      mouse.x = (event.clientX / this.renderer.domElement.clientWidth) * 2 - 1;
      mouse.y =
        -((event.clientY / this.renderer.domElement.clientHeight) * 2) + 1;
      return mouse;
    },
    onMouseClick() {},
    createLine(points) {
      const material = new THREE.LineBasicMaterial({
        color: 0x0000ff,
      });
      const geometry = new THREE.BufferGeometry().setFromPoints(points);

      const line = new THREE.Line(geometry, material);
      this.parentTransform.add(line);
      //   scene.add(line);
    },
    createLines() {
      const points = [];
      points.push([new THREE.Vector3(5, 0, 0), new THREE.Vector3(5, 5, 0)]);
      points.push([new THREE.Vector3(5, 5, 0), new THREE.Vector3(-5, 5, 0)]);
      points.push([new THREE.Vector3(-5, 5, 0), new THREE.Vector3(-5, 0, 0)]);
      points.push([new THREE.Vector3(-5, 0, 0), new THREE.Vector3(5, 0, 0)]);
      points.push([new THREE.Vector3(5, 0, 0), new THREE.Vector3(5, 0, 5)]);
      points.push([new THREE.Vector3(5, 0, 5), new THREE.Vector3(5, 5, 5)]);
      points.push([new THREE.Vector3(5, 5, 5), new THREE.Vector3(5, 5, 0)]);
      points.push([new THREE.Vector3(-5, 0, 0), new THREE.Vector3(-5, 0, 5)]);
      points.push([new THREE.Vector3(-5, 0, 5), new THREE.Vector3(-5, 5, 5)]);
      points.push([new THREE.Vector3(-5, 5, 5), new THREE.Vector3(-5, 5, 0)]);
      points.push([new THREE.Vector3(-5, 5, 5), new THREE.Vector3(5, 5, 5)]);
      points.push([new THREE.Vector3(5, 0, 5), new THREE.Vector3(-5, 0, 5)]);
      points.forEach((pointItem) => {
        this.createLine(pointItem);
      });
    },
    render() {
      this.renderer.clear();
      this.renderer.render(this.scene, this.camera);
    },
    animate() {
      requestAnimationFrame(this.animate);

      this.controls.update(); // only required if controls.enableDamping = true, or if controls.autoRotate = true

      this.renderer.render(this.scene, this.camera);
    },
  },
};
</script>
<style scoped>
#mainView {
  width: 80rem;
  height: 50rem;
}
</style>