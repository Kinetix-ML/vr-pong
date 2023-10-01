<script>
	import { T } from '@threlte/core';
	import * as THREE from 'three';
</script>

<T.PerspectiveCamera
	makeDefault
	position={[0, 2, 5]}
  fov=50
	on:create={({ ref }) => {
		ref.lookAt(0, 1, 0);
	}}
/>

<T.DirectionalLight position={[4, 6, 4]} castShadow />

<!-- Cube -->
<T.Mesh
  castShadow
  position={[2, 0, -1]}
>
	<T.BoxGeometry args={[1, 1, 2]} />
	<T.MeshStandardMaterial color="hotpink"/>
</T.Mesh>

<!-- Floor -->
<T.Mesh 
  receiveShadow 
  position={[0, -1, -1]}
>
  <T.BoxGeometry args={[100, 0.25, 100]}/>
  <T.MeshStandardMaterial color="white" />
</T.Mesh>