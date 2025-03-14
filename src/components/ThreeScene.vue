<script setup>
import { onMounted, ref } from 'vue';
import * as THREE from 'three';
import WebGL from 'three/addons/capabilities/WebGL.js';

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera( 75, window.innerWidth / window.innerHeight, 0.1, 1000 );
const renderer = ref(null);

onMounted(() => {
    if ( WebGL.isWebGL2Available() ) {
        renderer.value = new THREE.WebGLRenderer();
        renderer.value.setSize( window.innerWidth, window.innerHeight );
        document.body.appendChild( renderer.value.domElement );

        const geometry = new THREE.BoxGeometry();
        const material = new THREE.MeshBasicMaterial( { color: 0x00ff00 } );
        const cube = new THREE.Mesh( geometry, material );
        scene.add( cube );

        camera.position.z = 5;

        const animate = () => { 
            requestAnimationFrame( animate );
            cube.rotation.x += 0.01;
            cube.rotation.y += 0.01;
            renderer.value.render( scene, camera );
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