<script>
	import { T } from '@threlte/core';
</script>

<T.PerspectiveCamera
	makeDefault
	position={[10, 10, 10]}
	on:create={({ ref }) => {
		ref.lookAt(0, 1, 0);
	}}
/>

<T.Mesh>
	<T.BoxGeometry />
	<T.MeshBasicMaterial />
</T.Mesh>
