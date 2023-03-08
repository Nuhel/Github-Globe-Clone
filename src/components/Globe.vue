<script setup >
  import * as THREE from 'three';
  import {onMounted, ref} from "vue";
  import {OrbitControls} from "three/addons/controls/OrbitControls.js";
  import ThreeGlobe from "three-globe";
  import countries from '../assets/custom.geo.json'
  import line from '../assets/line.json'
  import map from '../assets/map.json'
  const threeComponent = ref();
  const threeComponentContainer = ref();


  let scene ;
  let camera;
  let renderer;
  let dLight1;
  let dLight2;
  let dLight3;
  let controls;
  let Globe;
  let height;
  let width;
  let mouseX;
  let mouseY;

  let ambientLight;
  onMounted(()=>{

    scene = new THREE.Scene()

     height = threeComponentContainer.value.offsetHeight;
     width = threeComponentContainer.value.offsetWidth;

    width = window.innerWidth
    height = window.innerHeight
    renderer = new THREE.WebGLRenderer();
    renderer.setSize( width, height );
    threeComponent.value.appendChild( renderer.domElement );

    //scene.fog = new THREE.Fog(0x535efe,400,2000)

    initCamera()
    initGlobe()
    initLights()

    initController();

    scene.background = new THREE.Color(0x040d21)
    window.addEventListener('resize',onWindowResize,false)
    document.addEventListener('mousemove',onMouseMove)
    animate();

  })

  const initGlobe = ()=>{

    const arcsData = [...Array(10).keys()].map((val,index) => ({
      startLat: (Math.random() - 0.5) * 180,
      startLng: (Math.random() - 0.5) * 360,
      endLat: (Math.random() - 0.5) * 180,
      endLng: (Math.random() - 0.5) * 360,
      color: ['red', 'white', 'blue', 'green'][Math.round(Math.random() * 3)],
      order: index

    }));
    const movementSpeed = 1500;
    const transitionTime = 0;
    let ringData = [];
    arcsData.map((data)=>{
      ringData.push({
        lat: data.endLat,
        lng: data.endLng,
        maxR: 5,
        propagationSpeed: -2,
        repeatPeriod: movementSpeed *2 + (200)
      });

    })

    Globe = new ThreeGlobe({
      waitForGlobeReady: true,
      animateIn: true
    })
        //.globeImageUrl(earth)
        .hexPolygonsData(countries.features)
        .hexPolygonResolution(3)
        .hexPolygonMargin(.5)
        .hexPolygonColor(()=>{
          return '#B2D3EC'
        })
        .showAtmosphere(true)
        .atmosphereColor('#3a228a')
        .atmosphereAltitude(.25)



    setTimeout(()=>{
      Globe.arcsData(arcsData)

          .ringsData(ringData)
          .ringMaxRadius('maxR')
          .ringPropagationSpeed('propagationSpeed')
          .ringRepeatPeriod('repeatPeriod')
          .arcColor((e)=>(e.color?? '#ff4000'))
          .arcAltitude((e)=>(.3))
          .arcStroke((e) => {
            return 1
          })

          .arcDashLength(1)
          .arcDashGap(1)
          .arcDashAnimateTime(movementSpeed)
          .arcsTransitionDuration(transitionTime)
          .arcDashInitialGap((e) => (e.order*1))
          .labelsData(map.coordinates)
          .labelColor(()=> "#ffcb21")

          .labelDotRadius(.03)
          .labelSize((e)=>e.size)
          .labelText('city')
          .labelResolution(10)
          .labelAltitude(.01)
          //pointsData(map.coordinates)
          .pointColor(()=>'#ffffff')
          .pointsMerge(true)
          .pointAltitude(.07)
          .pointRadius(.05)


    },100)

    Globe.rotateY(-Math.PI * 5/9)
    Globe.rotateY(-Math.PI * 6)
    const globeMaterial = Globe.globeMaterial();

    globeMaterial.color = new THREE.Color(0x3a228a)
    globeMaterial.emissive = new THREE.Color(0x220038)
    globeMaterial.emissiveIntensity = 0.1;
    globeMaterial.shineness = 0.7;
    scene.add(Globe)

  }

  const initController = ()=>{

    controls = new OrbitControls(camera, renderer.domElement);
    controls.autoRotate = true
    controls.enableDamping = true;
    controls.dynamicDampingFactor = .01
    controls.enablePan = false;
    controls.minDistance = 200
    controls.maxDistance = 500
    controls.rotateSpeed = .8
    controls.zoomSpeed = 1
    controls.minPolarAngle = Math.PI / 3.5;
    controls.maxPolarAngle = Math.PI - controls.minPolarAngle
  }

  const initCamera = () => {

    camera = new THREE.PerspectiveCamera( );
    camera.aspect = width / height
    camera.updateProjectionMatrix()
    camera.position.y = 0;
    camera.position.x = 0;
    camera.position.z = 400;

    scene.add(camera)


  }

  const initLights = ()=>{

    ambientLight = new THREE.AmbientLight(0xbbbbbb, 0.3)
    scene.add(ambientLight)

    dLight1 = new THREE.DirectionalLight(0xffffff, 0.8)
    dLight1.position.set(-800,2000,400)
    camera.add(dLight1)


    dLight2 = new THREE.DirectionalLight(0x7982f6, 1)
    dLight2.position.set(-200,500,200)
    camera.add(dLight2)


    dLight3 = new THREE.DirectionalLight(0x8566cc,.5)
    dLight3.position.set(-200,500,200)
    camera.add(dLight3)
  }

  const onWindowResize = ()=>{
    height = threeComponentContainer.value.offsetHeight;
    width = threeComponentContainer.value.offsetWidth;
    camera.aspect = width/height;
    camera.updateProjectionMatrix();
    renderer.setSize(width,height)
  }

  const onMouseMove = (event)=>{
    mouseX = event.clientX - (width/1.5)
    mouseY = event.clientY - (height/1.5)
  }

  const animate = ()=>{

    controls.update();
    renderer.render(scene, camera);
    requestAnimationFrame(animate);

  }

</script>

<template>

  <div ref="threeComponentContainer" class="max-w-full h-screen relative">
    <div ref="threeComponent" class="max-h-full max-w-full">
    </div>
  </div>



</template>

<style scoped>

</style>
