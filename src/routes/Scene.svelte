<script lang="ts">
	import { T, useFrame } from '@threlte/core';
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import type { KMLPipeline } from 'kml-pipe-ts';
	import type { Vec } from 'kml-pipe-ts/dist/types';
	let videoSource: HTMLVideoElement;
	let pipe: KMLPipeline;
	let cubeRef: THREE.Object3D;
	let cubeOrientation = [0, 0, 0];
	let cubeRotation = [0, 0, 0];
	let cubePosition = { x: 0, y: 0, z: 0 };

	let processing = false;

	onMount(async () => {
		let { KMLPipeline } = await import('kml-pipe-ts');
		pipe = new KMLPipeline('VR Pong', 1, '261e97fd-18b1-4a3c-8cc1-b94a421c11bf');
		await pipe.initialize();
		await startWebcam();
	});

	async function runInference() {
		if (!processing) {
			processing = true;
			let outputs = await pipe.execute([videoSource]);
			console.log(outputs);
			if (outputs[1].value && outputs[0].value) {
				console.log('updating pose');
				cubeOrientation = outputs[1].value as Vec;
				let z = Math.atan(cubeOrientation[0] / cubeOrientation[1]);
				let x = Math.atan(cubeOrientation[2] / cubeOrientation[1]);
				console.log(z);
				cubeRotation = [x, 0, z];
				cubePosition = outputs[0].value;
			}
			processing = false;
		}
		videoSource.requestVideoFrameCallback(runInference); // run inference on every frame
	}

	const { start, stop, started } = useFrame(
		() => {
			console.log('rendering…');
			//cubeRef.lookAt(new THREE.Vector3(...cubeOrientation));
		},
		{
			autostart: false
		}
	);

	async function startWebcam() {
		const stream = await navigator.mediaDevices.getUserMedia({ video: true });
		videoSource.srcObject = stream;
		await videoSource.play();
		videoSource.requestVideoFrameCallback(runInference);
		start();
	}
</script>

<T.PerspectiveCamera
	makeDefault
	position={[0, 1, 1]}
	fov="50"
	on:create={({ ref }) => {
		ref.lookAt(0, 1, 0);
	}}
/>

<T.DirectionalLight position={[4, 6, 4]} castShadow />

<!-- Cube -->
<T.Mesh
	castShadow
	position={[-cubePosition.x, cubePosition.y, cubePosition.z]}
	rotation={cubeRotation}
	on:create={({ ref }) => {
		cubeRef = ref;
	}}
>
	<T.BoxGeometry args={[0.1, 0.2, 0.1]} />
	<T.MeshStandardMaterial color="hotpink" />
</T.Mesh>

<!-- Floor -->
<!-- <T.Mesh receiveShadow position={[0, -1, -1]}>
	<T.BoxGeometry args={[100, 0.25, 100]} />
	<T.MeshStandardMaterial color="white" />
</T.Mesh> -->

<video autoplay bind:this={videoSource} />
