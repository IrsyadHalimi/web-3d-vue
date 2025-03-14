<script setup>
import { onMounted, ref, render } from 'vue';
import * as THREE from 'three';
import WebGL from 'three/addons/capabilities/WebGL.js';
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
import gsap from "gsap";

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
const renderer = ref(null);
let mixer;

onMounted(() => {
    if ( WebGL.isWebGL2Available() ) {
        renderer.value = new THREE.WebGLRenderer({ antialias: true });
        renderer.value.setSize( window.innerWidth, window.innerHeight );
        renderer.value.shadowMap.enabled = true;
        document.body.appendChild( renderer.value.domElement );

        const light = new THREE.DirectionalLight( 0xffffff, 1 );
        light.position.set( 5, 10, 5 );
        light.castShadow = true;
        scene.add( light );

        const ambientLight = new THREE.AmbientLight( 0x404040, 2 );
        scene.add( ambientLight );

        const floorGeometry = new THREE.PlaneGeometry(20, 20);
        const floorMaterial = new THREE.MeshStandardMaterial({ color: 0x999999 });
        const floor = new THREE.Mesh(floorGeometry, floorMaterial);
        floor.rotation.x = -Math.PI / 2;
        floor.receiveShadow = true;
        scene.add(floor);

        const loader = new GLTFLoader();
        loader.load(
            "/models/character.glb", (gltf) => {
                const model = gltf.scene;
                model.position.set(0, 0, 0);
                model.castShadow = true;
                scene.add(model);
                mixer = new THREE.AnimationMixer(model);
                if (gltf.animations.length) {
                    const action = mixer.clipAction(gltf.animations[0]);
                    action.play();
                }
            }
        );

        camera.position.set(0, 2, 5);

        gsap.to(camera.position, {
            z: 3, duration: 3, ease: "power2.out"
        });

        window.addEventListener("click", (event) => {
            const mouse = new THREE.Vector2(
                (event.clientX / window.innerWidth) * 2 - 1,
                -(event.clientY / window.innerHeight) * 2 + 1
            );

            const raycaster = new THREE.Raycaster();
            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects(scene.children);

            if (intersects.length > 0) {
                gsap.to(intersects[0].object.position, { y: "+=0.5", duration: 0.5, yoyo: true, repeat: 1 });
            }
        });

        const keys = {};
        window.addEventListener("keydown", (event) => (keys[event.key] = true));
        window.addEventListener("keyup", (event) => (keys[event.key] = false));
        
        function animate() {
            requestAnimationFrame(animate);
            if (mixer) mixer.update(0.1);
            if (keys["w"]) camera.position.z -= 0.1;
            if (keys["s"]) camera.position.z += 0.1;
            if (keys["a"]) camera.position.x -= 0.1;
            if (keys["d"]) camera.position.x += 0.1;
            renderer.value.render(scene, camera);
        }
        animate();
    } else {
        const warning = WebGL.getWebGL2ErrorMessage();
        document.getElementById( 'container' ).appendChild( warning );
    }
});
</script>

<template>
    <div id="container"></div>
    <div id="info">Description</div>
</template>

<style scoped>
#info {
    position: absolute;
    top: 10px;
    Width: 100%;
    text-align: center;
    z-index: 100;
    display: block;
}
</style>