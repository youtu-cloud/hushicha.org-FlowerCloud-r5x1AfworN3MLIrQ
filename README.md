
在 360° 看房功能中，我们需要在浏览器中创建一个类似虚拟现实的场景，使得用户能够查看环境的每一个角落。这一功能的实现本质上是利用 球体映射技术，即通过将全景图作为纹理贴图映射到一个反向的球体上，用户可以通过旋转视角来“环顾四周”。


我们先来看一下效果
!\[file](Maximum upload size exceeded; nested exception is java.lang.IllegalStateException: org.apache.tomcat.util.http.fileupload.FileUploadBase$SizeLimitExceededException: the request was rejected because its size (4167362\) exceeds the configured maximum (2097152\))


## 步骤拆解


#### 创建球体模型


为了让用户能够查看 360° 环境，我们需要构建一个大球体。通过将球体的内表面作为视图区域，用户的相机将被放置在球体的中心。


球体内表面可见：通过将球体的外表面设置为不可见，而把全景图像贴图到球体的内侧，使得相机站在球体的中心时，所有视野范围都被全景图包围。


#### 全景图像作为纹理贴图


全景图是一种特殊的图像格式，通常是水平或垂直方向无缝拼接的图像，通常为环状。我们可以将全景图作为纹理，映射到球体的内表面。


映射全景图：全景图的每个像素会映射到球体表面的某个位置，形成一个 3D 场景，用户通过旋转相机来查看不同的部分。


#### 相机位置与控制


为了让用户从球体的中心观察全景图，必须将相机的位置设置在球体的内部。同时，我们需要监听用户的输入（如鼠标或触控）来旋转视角。


相机控制：通过旋转相机的方位角（通常是使用 lon 和 lat）来改变用户的视角，从而模拟 360° 环视的效果。


#### 球体的反向显示


由于默认情况下，Three.js 的球体表面是朝外的，因此我们需要将球体反转，让其内表面可见。这通过 geometry.scale(\-1, 1, 1\) 来实现。


## 创建 360° 看房 Demo


下面，我们将按照以下步骤一步步实现一个简易的 360° 看房功能。


### 项目准备


#### 全景图下载


可以通过网站 [Poly Haven](https://github.com) 下载，这是一个提供免费高质量纹理的网站。我们案例通过
[https://polyhaven.com/a/hotel\_room](https://github.com):[westworld加速器](https://xbsj9.com) 进行下载
![](image/360.png)


#### 安装和引入依赖


在项目中，你需要通过 npm 安装 Three.js，并且确保你的项目支持使用 ES Modules（现代 JavaScript 模块）。



```
npm install three

```

然后在你的 JavaScript 文件中引入 Three.js 和 EXRLoader：



```
import * as THREE from "three";
import { EXRLoader } from "three/examples/jsm/loaders/EXRLoader.js";
import "./style.css"; // 样式文件

```

在 style 中，我们就写了很简单的 css。只需要把默认的 margin 和 padding 去掉就行



```
* {
  margin: 0;
  padding: 0;
}

```

### 创建场景和相机


在 Three.js 中，所有的图形和模型都需要添加到 **场景** 中，而 **相机** 用来决定我们看到场景的视角。这里我们创建一个简单的 **透视相机**，并设置视野为 75 度，近裁剪面为 0\.1，远裁剪面为 1000。



```
// 场景
const scene = new THREE.Scene();

// 相机：视角设置为 75 度，近裁剪面 0.1，远裁剪面 1000
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 0, 0); // 将相机位置设置在中心点

```

* THREE.Scene()：创建一个 3D 场景容器，所有的物体、光源、相机都会添加到其中。
* THREE.PerspectiveCamera()：创建一个透视相机，设置视角为 75 度。该相机会模拟人眼的视觉效果，适合于大多数场景。
* THREE.WebGLRenderer()：创建 WebGL 渲染器，用来将 Three.js 场景渲染到浏览器的 canvas 元素中。


### 创建渲染器


使用 THREE.WebGLRenderer 渲染器来渲染三维场景，并将其显示在浏览器窗口中。你还需要设置渲染器的大小，并将其挂载到 HTML 页面上。



```
// 渲染器
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

```

### 创建球体并应用全景图纹理


我们将使用一个球体（THREE.SphereGeometry）来显示全景图。需要特别注意的是，为了将全景图映射到球体的内表面，我们需要将球体的几何体进行反向缩放，即 geometry.scale(\-1, 1, 1\)。


然后，使用 EXRLoader 加载 .exr 格式的全景图，并将其应用到球体的材质中。


#### 创建球体



```
const geometry = new THREE.SphereGeometry(500, 60, 40); // 半径 500，60 段宽度，40 段高度

```

#### 球体反向


为了将全景图贴到球体内部，我们需要将球体反向显示。这是因为 **全景图是通过球体的内表面来展示的**，而相机需要位于球体内部。



```
geometry.scale(-1, 1, 1); // 反向缩放球体，使其内表面可见

```

这行代码将球体的 **x 轴** 缩放为 `-1`，使球体的内表面朝向相机。这样，相机就位于球体内部，可以查看到全景图的内容。


#### 加载全景图纹理


在 Three.js 中，纹理是通过 **Loader** 加载的。由于我们加载的是 `.exr` 格式的全景图，我们需要使用 `EXRLoader` 来加载纹理。



```
const texture = new EXRLoader().load("hotel_room_4k.exr");

```

这里，`"hotel_room_4k.exr"` 是我们房屋的全景图,将其放到 public 文件夹下面，这张图片也是就是我们刚才下载的那张图。你可以根据需要替换为其他全景图文件。加载纹理后，Three.js 会自动将纹理映射到球体的内表面。


#### 创建材质并将纹理应用到球体


在 Three.js 中，材质是与几何体一起使用的，它定义了物体表面的外观。在这里，我们使用 `THREE.MeshBasicMaterial` 来创建一个基本材质，并将加载的纹理应用到材质上。



```
const material = new THREE.MeshBasicMaterial({ map: texture });

```

`THREE.MeshBasicMaterial` 是一种不受光照影响的材质，适合用于全景图等无需光照的场景。`map` 属性用于设置纹理，我们把加载的 `texture` 赋值给了它。


#### 创建球体并加入场景


接下来，我们将创建一个 **网格（Mesh）**，它是由几何体和材质组成的。我们用球体几何体和材质创建网格，并将其添加到场景中。



```
const sphere = new THREE.Mesh(geometry, material);
scene.add(sphere);

```

通过这行代码，球体与材质组合成了一个网格，并且这个网格被添加到场景中。这样，球体就成为了我们展示全景图的基础物体。


我们现临时增加这段代码，用于阶段性的测试。将上面的内容渲染到页面上。



```
function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}

animate();

```

!\[file](Maximum upload size exceeded; nested exception is java.lang.IllegalStateException: org.apache.tomcat.util.http.fileupload.FileUploadBase$SizeLimitExceededException: the request was rejected because its size (2768870\) exceeds the configured maximum (2097152\))


### 相机控制与交互


为了让用户能够通过鼠标拖动来查看全景图的不同角度，我们需要实现 **相机控制与交互**。用户通过鼠标的拖动操作来改变相机的视角，我们会根据鼠标的移动来更新相机的经度（lon）和纬度（lat），然后重新计算相机的位置。


#### 记录鼠标按下位置


首先，我们需要记录鼠标按下的初始位置。当用户按下鼠标时，我们需要记录下鼠标当前位置的 **X 坐标** 和 **Y 坐标**，以及当前相机的 **经度（lon）** 和 **纬度（lat）**。这些值将用于计算相机在鼠标拖动过程中应该旋转的角度。



```
let lon = 0,
  lat = 0; // 初始化经度和纬度
let isUserInteracting = false; // 标记是否正在进行交互
let onPointerDownPointerX = 0,
  onPointerDownPointerY = 0;
let onPointerDownLon = 0,
  onPointerDownLat = 0;

// 鼠标按下事件
document.addEventListener("pointerdown", (event) => {
  isUserInteracting = true; // 用户正在交互
  onPointerDownPointerX = event.clientX; // 记录鼠标按下时的X坐标
  onPointerDownPointerY = event.clientY; // 记录鼠标按下时的Y坐标
  onPointerDownLon = lon; // 记录当前经度
  onPointerDownLat = lat; // 记录当前纬度
});

```

#### 计算鼠标移动


在鼠标拖动的过程中，我们需要根据鼠标移动的距离来更新相机的 **经度（lon）** 和 **纬度（lat）**。通过比较当前鼠标的位置和按下时的位置的差异，我们可以计算出相机应该旋转的角度。



```
document.addEventListener("pointermove", (event) => {
  if (isUserInteracting) {
    lon = (onPointerDownPointerX - event.clientX) * 0.1 + onPointerDownLon;
    lat = (event.clientY - onPointerDownPointerY) * 0.1 + onPointerDownLat;
  }
});

```

* `lon` 是经度，表示水平旋转的角度。
* `lat` 是纬度，表示垂直旋转的角度。


通过这段代码，我们根据鼠标移动的水平和垂直距离，更新经度和纬度的值，从而改变相机的朝向。


#### 限制纬度范围


为了避免用户将视角旋转到不合理的位置，我们需要限制 **纬度（lat）** 的范围。纬度的范围应该在 **\-85° 到 85° 之间**，以防用户将视角转到地球的极端位置（超过 85° 或 \-85° 会导致不可见的效果）。



```
lat = Math.max(-85, Math.min(85, lat)); // 限制纬度的范围

```

#### 更新相机的位置


当用户拖动鼠标时，我们根据经度和纬度来计算相机的新位置。相机的位置应该与球体中心（即场景的原点）保持一定的距离，并朝向球体的中心。



```
const phi = THREE.MathUtils.degToRad(90 - lat); // 将纬度转为弧度
const theta = THREE.MathUtils.degToRad(lon); // 将经度转为弧度

// 计算相机的位置
const lookAtX = 500 * Math.sin(phi) * Math.cos(theta);
const lookAtY = 500 * Math.cos(phi);
const lookAtZ = 500 * Math.sin(phi) * Math.sin(theta);

// 使相机始终朝向场景的中心
camera.lookAt(lookAtX, lookAtY, lookAtZ);

```

* `phi` 是纬度角度转化成的弧度，`theta` 是经度角度转化成的弧度。
* 我们通过这些角度来计算相机的 **x**, **y**, **z** 位置，确保相机始终朝向球体的中心（即全景图的中心）。


### 动画与渲染


通过 requestAnimationFrame 来创建动画循环，不断更新相机的朝向并渲染场景。每帧都需要计算相机的朝向，并确保用户不会在纬度方向上超出可视范围（\-85 到 85）。



```
// 动画循环
function animate() {
  requestAnimationFrame(animate);

  // 限制角度范围，防止旋转过度
  lat = Math.max(-85, Math.min(85, lat)); // 只限制lat的范围

  const phi = THREE.MathUtils.degToRad(90 - lat); // 使用 Three.js 的角度转弧度方法
  const theta = THREE.MathUtils.degToRad(lon);

  // 计算相机的朝向
  const lookAtX = 500 * Math.sin(phi) * Math.cos(theta);
  const lookAtY = 500 * Math.cos(phi);
  const lookAtZ = 500 * Math.sin(phi) * Math.sin(theta);

  camera.lookAt(lookAtX, lookAtY, lookAtZ);

  renderer.render(scene, camera);
}

// 启动动画
animate();

```

### 窗口大小调整


为了确保在用户调整浏览器窗口大小时，场景能够自动适配新的大小，我们监听了窗口的 resize 事件，并更新渲染器的尺寸与相机的纵横比。



```
// 窗口大小调整事件
window.addEventListener("resize", () => {
  renderer.setSize(window.innerWidth, window.innerHeight);
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
});

```

到此为止，你就可以看到一个完整的 360° 看房功能。用户可以通过鼠标拖动来查看全景图的不同角度，实现了一个简单的 360° 看房效果。


效果如我们一开始所展示的那样，用户可以通过鼠标拖动来查看房屋的各个角落。


Three.js学习：[https://www.threejs3d.com/](https://github.com)


