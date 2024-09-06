<template></template>

<script setup>
import { onMounted } from 'vue';
import * as BABYLON from 'babylonjs';



function getTriangles(mesh) {
  const positions = mesh.getVerticesData(BABYLON.VertexBuffer.PositionKind);
  const indices = mesh.getIndices();
  const worldMatrix = mesh.getWorldMatrix();
  const triangles = [];

  for (let i = 0; i < indices.length; i += 3) {
    const v1 = BABYLON.Vector3.TransformCoordinates(BABYLON.Vector3.FromArray(positions, indices[i] * 3), worldMatrix);
    const v2 = BABYLON.Vector3.TransformCoordinates(BABYLON.Vector3.FromArray(positions, indices[i + 1] * 3), worldMatrix);
    const v3 = BABYLON.Vector3.TransformCoordinates(BABYLON.Vector3.FromArray(positions, indices[i + 2] * 3), worldMatrix);

    triangles.push({ vertices: [v1, v2, v3], indices: [indices[i], indices[i + 1], indices[i + 2]] });
  }

  return triangles;
}


// Helper function to check if a vertex is inside another cube's bounding box
function isVertexInsideCube(vertex, cube) {
  const boundingBox = cube.getBoundingInfo().boundingBox;
  const min = boundingBox.minimumWorld;
  const max = boundingBox.maximumWorld;

  // Check if the vertex is within the bounding box's bounds
  return (
    vertex.x >= min.x && vertex.x <= max.x &&
    vertex.y >= min.y && vertex.y <= max.y &&
    vertex.z >= min.z && vertex.z <= max.z
  );
}

onMounted(() => {
  const babylonCanvas = document.createElement('canvas');
  document.getElementById('app').appendChild(babylonCanvas);
  const engine = new BABYLON.Engine(babylonCanvas, true);

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
  scene.clearColor = new BABYLON.Color4(0, 0, 0, 1);

  const box1 = BABYLON.MeshBuilder.CreateBox('box1', { size: 1 }, scene);
  box1.position = new BABYLON.Vector3(0, 0, 0);

  const box2 = BABYLON.MeshBuilder.CreateBox('box2', { size: 1.25 }, scene);
  box2.position = new BABYLON.Vector3(0.5, -0.5, 0);


  const triangles = getTriangles(box1);

  // Check if vertices of cube1 are inside cube2
  triangles.forEach((triangle, idx) => {
    const [v1, v2, v3] = triangle.vertices;

    const isV1Inside = isVertexInsideCube(v1, box2);
    const isV2Inside = isVertexInsideCube(v2, box2);
    const isV3Inside = isVertexInsideCube(v3, box2);

    console.log(`Triangle ${idx + 1}: V1 inside box2:`, isV1Inside);
    console.log(`Triangle ${idx + 1}: V2 inside box2:`, isV2Inside);
    console.log(`Triangle ${idx + 1}: V3 inside box2:`, isV3Inside);
  });



  engine.runRenderLoop(() => {
    scene.render();
  });

  window.addEventListener('resize', () => {
    engine.resize();
  });
});
</script>
