<script lang="ts">
	import { T, useTask } from '@threlte/core'
	import {
		Gizmo,
		Instance,
		InstancedMesh,
		interactivity,
		OrbitControls,
		useCursor,
		useInteractivity
	} from '@threlte/extras'
	import { spring } from 'svelte/motion'
	import * as THREE from 'three'

	// IX
	interactivity()
	const ixSpring = spring(1, { stiffness: 0.125, damping: 0.125 })

	// ORBIT
	const { pointer, pointerOverTarget } = useInteractivity()

	let orbitControlsRef: any = null
	let targetRotation = 0
	const rotationSpeed = 0.1 // Adjust this value to change rotation speed

	$: {
		if (orbitControlsRef) {
			targetRotation = $pointer.x * Math.PI * 0.5 // Map pointer.x (-1 to 1) to rotation (-PI to PI)
		}
	}

	$: {
		if (!$pointerOverTarget) {
			targetRotation = 0
		}
	}

	// COLOR HOVER
	const { hovering, onPointerEnter, onPointerLeave } = useCursor()
	$: localClr = $hovering ? '#ffa0be' : '#00a3ff'

	$: console.log(localClr)

	// TASK
	let spin = 0
	useTask((delta) => {
		spin += delta

		if (orbitControlsRef && orbitControlsRef.object) {
			const camera = orbitControlsRef.object as THREE.Camera

			// Calculate how far we want to rotate based on mouse position
			const rotationAmount = $pointer.x * Math.PI * 0.333

			// Get the current distance from the center
			const distance = camera.position.length()

			// Calculate new X and Z positions
			camera.position.x = Math.sin(rotationAmount) * distance
			camera.position.z = Math.cos(rotationAmount) * distance

			orbitControlsRef.update()
		}
	})
</script>

<!-- MARKUP -->
<Gizmo size={80}></Gizmo>

<!-- Camera -->
<!-- <T.PerspectiveCamera
	makeDefault
	position={[0, 0, cameraZ]}
	fov={100}
	on:create={({ ref }) => {
		ref.lookAt(0, 0, 0);
	}}
/> -->

<T.OrthographicCamera
	makeDefault
	position={[0, 0, 12]}
	zoom={50}
	on:create={({ ref }) => {
		ref.lookAt(0, 0, 0)
	}}
>
	<OrbitControls
		enableZoom={false}
		enablePan={false}
		enableRotate={false}
		autoRotate={false}
		minPolarAngle={Math.PI / 2}
		maxPolarAngle={Math.PI / 2}
		on:create={({ ref }) => {
			orbitControlsRef = ref
		}}
	/>
</T.OrthographicCamera>

<!-- Lighting -->
<T.DirectionalLight intensity={2} position={[-2, -2, 10]} />
<T.AmbientLight intensity={4} color={'#ff00ff'} />

<!-- Mesh -->
<T.Group on:pointerenter={() => ixSpring.set(1.25)} on:pointerleave={() => ixSpring.set(1)}>
	<InstancedMesh scale={$ixSpring} rotation.z={spin}>
		<T.IcosahedronGeometry args={[1, 0]} />
		<T.MeshPhysicalMaterial
			color={localClr}
			metalness={0.5}
			roughness={0.5}
			clearcoat={1}
			clearcoatRoughness={0.1}
		/>
		<!--  -->
		<Instance
			position={[0, 1.5, 0]}
			rotation={[spin, spin, spin]}
			on:pointerenter={onPointerEnter}
			on:pointerleave={onPointerLeave}
		/>
		<Instance
			position={[1.5, 0, 0]}
			rotation={[spin, spin, spin]}
			on:pointerenter={onPointerEnter}
			on:pointerleave={onPointerLeave}
		/>
		<Instance
			position={[0, -1.5, 0]}
			rotation={[spin, spin, spin]}
			on:pointerenter={onPointerEnter}
			on:pointerleave={onPointerLeave}
		/>
		<Instance
			position={[-1.5, 0, 0]}
			rotation={[spin, spin, spin]}
			on:pointerenter={onPointerEnter}
			on:pointerleave={onPointerLeave}
		/>
	</InstancedMesh>
</T.Group>
