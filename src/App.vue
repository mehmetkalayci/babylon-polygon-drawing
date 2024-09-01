<template>
  <div id="babylon"></div>
  <svg id="svg"></svg>
</template>

<script setup>
import { onMounted } from 'vue';
import * as BABYLON from 'babylonjs';

onMounted(() => {
    // Create a basic Babylon.js scene
    const babylonCanvas = document.createElement('canvas');
    document.getElementById('babylon').appendChild(babylonCanvas);
    const engine = new BABYLON.Engine(babylonCanvas, true);

    const createScene = function () {
        const scene = new BABYLON.Scene(engine);
        const camera = new BABYLON.ArcRotateCamera(
            'camera1',
            Math.PI / 2, Math.PI / 4,
            10,
            BABYLON.Vector3.Zero(),
            scene
        );
        camera.attachControl(babylonCanvas, true);

        const light = new BABYLON.HemisphericLight('light1', new BABYLON.Vector3(1, 1, 0), scene);

        // Create a box
        const box = BABYLON.MeshBuilder.CreateBox('box', { size: 2 }, scene);

        // Set the background color
        scene.clearColor = new BABYLON.Color4(0, 0, 0, 1);

        return { scene, box, camera };
    };

    const { scene, box, camera } = createScene();
    engine.runRenderLoop(() => {
        scene.render();
        updatePolygon();
    });

    // Resize event
    window.addEventListener('resize', () => {
        engine.resize();
    });

    const svg = document.getElementById('svg');
    const polygon = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
    polygon.setAttribute('fill', 'rgba(255, 0, 0, 0.5)');
    polygon.setAttribute('stroke', 'black');
    polygon.setAttribute('stroke-width', '2');
    svg.appendChild(polygon);

    function updatePolygon() {
        // Retrieve the vertex data
        const positions = box.getVerticesData(BABYLON.VertexBuffer.PositionKind);
        
        let points = '';

        // Project 3D positions into 2D space
        for (let i = 0; i < positions.length; i += 3) {
            const vertex = BABYLON.Vector3.FromArray(positions, i);
            const projectedVertex = BABYLON.Vector3.Project(
                vertex,
                BABYLON.Matrix.Identity(),
                scene.getTransformMatrix(),
                camera.viewport.toGlobal(engine.getRenderWidth(), engine.getRenderHeight())
            );

            const x = projectedVertex.x;
            const y = projectedVertex.y;

            points += `${x},${y} `;
        }

        // Update the SVG polygon points
        polygon.setAttribute('points', points.trim());
    }
});
</script>

<style>
#app {
  display: flex;
}
canvas, svg {
  width: 600px;
  height: 600px;
}
#svg {
  background-color: rgba(255, 2, 2, 0.1);
}
</style>
