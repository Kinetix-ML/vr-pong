<script>
	import { T } from '@threlte/core';
	import * as THREE from 'three';
</script>

<T.PerspectiveCamera
	makeDefault
	position={[0, 5, 10]}
	on:create={({ ref }) => {
		ref.lookAt(0, 0, 0);
	}}
/>

<T.SpotLight
	position={[-40, 40, -15]}
	castShadow
	on:create={({ ref }) => {
		ref.castShadow = true;
		ref.shadow.mapSize = new THREE.Vector2(1024, 1024);
		ref.shadow.camera.far = 130;
		ref.shadow.camera.near = 40;
	}}
/>

<T.DirectionalLight position={[0, 5, 5]} castShadow />
<T.Mesh
	castShadow
	receiveShadow
	on:create={({ ref }) => {
		ref.lookAt(0, 1, 0);
	}}
>
	<T.BoxGeometry args={[1, 1, 2]} />
	<T.MeshBasicMaterial />
</T.Mesh>
