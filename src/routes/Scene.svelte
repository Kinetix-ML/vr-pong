<script lang="ts">
	import { T } from '@threlte/core';
	import { onMount } from 'svelte';
	import * as THREE from 'three';
	import type { KMLPipeline } from 'kml-pipe-ts';
	import type { Vec } from 'kml-pipe-ts/dist/types';
	let videoSource: HTMLVideoElement;
	let pipe: KMLPipeline;
	let cubeRef: THREE.Object3D;
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
			if (outputs[1].value) {
				console.log('updating pose');
				cubeRef.lookAt(new THREE.Vector3(...(outputs[1].value as Vec)));
			}
			processing = false;
		}
		videoSource.requestVideoFrameCallback(runInference); // run inference on every frame
	}

	async function startWebcam() {
		const stream = await navigator.mediaDevices.getUserMedia({ video: true });
		videoSource.srcObject = stream;
		await videoSource.play();
		videoSource.requestVideoFrameCallback(runInference);
	}
</script>

<T.PerspectiveCamera
	makeDefault
	position={[0, -2, 5]}
	fov="50"
	on:create={({ ref }) => {
		ref.lookAt(0, -2, 0);
	}}
/>

<T.DirectionalLight position={[4, 6, 4]} castShadow />

<!-- Cube -->
<T.Mesh
	castShadow
	position={[0, -2, 0]}
	on:create={({ ref }) => {
		cubeRef = ref;
	}}
>
	<T.BoxGeometry args={[1, 1, 2]} />
	<T.MeshStandardMaterial color="hotpink" />
</T.Mesh>

<!-- Floor -->
<!-- <T.Mesh receiveShadow position={[0, -1, -1]}>
	<T.BoxGeometry args={[100, 0.25, 100]} />
	<T.MeshStandardMaterial color="white" />
</T.Mesh> -->

<video autoplay bind:this={videoSource} />
