<script lang="ts">
	import { T, useFrame, useThrelte } from '@threlte/core';
	import { OrbitControls } from '@threlte/extras'
	import { onMount } from 'svelte';
	import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
	import { Group, TextureLoader } from 'three'; 
	import type { KMLPipeline } from 'kml-pipe-ts';
	import type { Vec } from 'kml-pipe-ts/dist/types';
	import { type SkinnedMesh, MeshStandardMaterial, Vector2, Box3 } from 'three';
	import { RigidBody, Collider, Debug } from '@threlte/rapier';


	let videoSource: HTMLVideoElement;
	let pipe: KMLPipeline;

	let paddleGroupRef: Group;
	let oPaddleGroupRef: Group;

	let paddleRef: SkinnedMesh;
	let oPaddleRef: SkinnedMesh;

	let ballRef: THREE.Object3D;
	let paddeOrientation = [0, 0, 0];
	let paddleRotation = [0, 0, 0];
	let paddePosition = { x: 1, y: 1, z: 2 };
	let paddleSize: [hx: number, hy: number, hz: number] = [0.125, 0.2, 0.017]

	let beganSet = false

	let paddleModel: SkinnedMesh | undefined;

	const { scene } = useThrelte();

	new GLTFLoader().load('paddle.glb', mesh => {
		paddleRef = mesh.scene.children[0].clone() as SkinnedMesh
		oPaddleRef = mesh.scene.children[0].clone() as SkinnedMesh

		let box = new Box3().setFromObject(paddleRef)
		let scale = paddleRef.scale.x*(0.25/(box.max.x - box.min.x))

		paddleRef.scale.set(scale, scale, scale)
		oPaddleRef.scale.set(scale, scale, scale)

		let tLoader = new TextureLoader()
		
		tLoader.load('low_Paddle_BaseColor.jpg', diffuse => {
			tLoader.load('low_Paddle_Metallic.jpg', metalic => {
				tLoader.load('low_Paddle_Normal.jpg', normal => {
					tLoader.load('low_Paddle_Roughness.jpg', roughness => {
						diffuse.flipY = false
						metalic.flipY = false
						normal.flipY = false
						roughness.flipY = false

						const mat = new MeshStandardMaterial({
							map: diffuse,
							normalMap: normal,
							metalnessMap: metalic,
							roughnessMap: roughness
						})

						paddleRef.material = mat
						oPaddleRef.material = mat

						paddleRef.position.set(0,-0.2,0)
						oPaddleRef.position.set(0,-0.2,0)

						paddePosition = { x: 0.75, y: 1, z: 2 }

						paddleGroupRef.children[0].add(paddleRef)
						oPaddleGroupRef.children[0].add(oPaddleRef)
					})
				})
			})
		})
	})

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
			// console.log(outputs);
			if (outputs[1].value && outputs[0].value) {
				console.log('updating pose');
				paddeOrientation = outputs[1].value as Vec;
				let z = Math.atan(paddeOrientation[0] / paddeOrientation[1]);
				let x = Math.atan(paddeOrientation[2] / paddeOrientation[1]);
				console.log(z);
				paddleRotation = [x, 0, z];
				paddePosition = outputs[0].value;
			}
			processing = false;
		}
		videoSource.requestVideoFrameCallback(runInference); // run inference on every frame
	}

	const { start, stop, started } = useFrame(
		(_, delta) => {
			console.log('rendering…');

			// cubeRef.lookAt(new THREE.Vector3(...cubeOrientation));
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
	position={[-1, 1.4, 4]}
	fov="70"
	on:create={({ ref }) => {
		ref.lookAt(0, 0, 0);
	}}
>
	<OrbitControls/>
</T.PerspectiveCamera>

<T.DirectionalLight position={[4, 6, 4]} castShadow />
<T.AmbientLight />

<!-- Ball -->
<T.Group
	position={[paddePosition.x,paddePosition.y+0.1,paddePosition.z-0.2]}
>
<RigidBody type='dynamic'>
<T.Mesh
	castShadow
	on:create={({ ref }) => {
		ballRef = ref;
	}}
>
<T.SphereGeometry args={[0.03]} />
<T.MeshStandardMaterial color="yellowgreen" />
</T.Mesh>

<Collider
	restitution={0.9}
	shape={'ball'}
	args={[0.03]}
	mass={beganSet ? 1 : Infinity}
/>
</RigidBody>
</T.Group>

<!-- Table -->
<T.Group
	position={[0,0,0]}
>
<RigidBody type='fixed'>
<T.Mesh castShadow>
	<T.BoxGeometry args={[2.5, 1, 4.5]} />
	<T.MeshStandardMaterial color="hotpink" />
</T.Mesh>

<Collider
	restitution={0.6}
	args={[1.25,0.5,2.25]}
	shape={'cuboid'}
/>
</RigidBody>
</T.Group>

<!-- Net -->
<T.Group
	position={[0,0.575,0]}
>
<RigidBody type='fixed'>
<T.Mesh
	castShadow
>
	<T.BoxGeometry args={[3, 0.35, 0.01]} />
	<T.MeshStandardMaterial color="hotpink" />
</T.Mesh>

<Collider
	restitution={-0.3}
	args={[1.5,0.175,0.005]}
	shape={'cuboid'}
/>
</RigidBody>
</T.Group>

<!-- Player Paddle -->

<T.Group
position={[paddePosition.x, paddePosition.y, paddePosition.z]}
orientation={paddeOrientation}
on:create={({ref}) => paddleGroupRef = ref}
>
<RigidBody type='dynamic'>
	
	{#if paddleGroupRef && paddleRef}
	<Collider
	restitution={0.7}
	args={paddleSize}
	shape={'cuboid'}
	mass={Infinity}
	/>
	{/if}

</RigidBody>
</T.Group>

<!-- Other Paddle -->
<T.Group
position={[-paddePosition.x, paddePosition.y, -paddePosition.z]}
orientation={paddeOrientation}
on:create={({ref}) => oPaddleGroupRef = ref}
>
<RigidBody type='dynamic'>
	
	{#if oPaddleGroupRef && oPaddleRef}
	<Collider
	restitution={0.7}
	args={paddleSize}
	shape={'cuboid'}
	mass={Infinity}
	/>
	{/if}

</RigidBody>
</T.Group>

<!-- Floor -->
<T.Mesh receiveShadow position={[0, -0.5, -1]}>
	<T.BoxGeometry args={[100, 0.25, 100]} />
	<T.MeshStandardMaterial color="gray" />
</T.Mesh>

<video autoplay bind:this={videoSource} />

<Debug />