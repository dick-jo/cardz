<script lang="ts">
	import { T, useTask, useThrelte } from '@threlte/core'
	import {
		Environment,
		Gizmo,
		interactivity,
		OrbitControls,
		useInteractivity
	} from '@threlte/extras'
	import { spring } from 'svelte/motion'
	import * as THREE from 'three'
	import CardPackTex from './models/CardPackTex.svelte'

	// SCENE ---------------------------------- //
	const { scene } = useThrelte()

	// DEBUG ---------------------------------- //
	let directionalLightHelper: THREE.DirectionalLightHelper

	// IX ------------------------------------- //
	interactivity()
	const { pointer, pointerOverTarget } = useInteractivity()

	let orbitControlsRef: any = null
	const ORBIT_ROT_CLAMP = 0.125
	const orbitSpring = spring(0.33, {
		stiffness: 0.125,
		damping: 0.25
	})

	$: {
		if (orbitControlsRef) {
			orbitSpring.set($pointer.x * Math.PI * ORBIT_ROT_CLAMP)
		}

		if (!$pointerOverTarget) {
			orbitSpring.set(0.33)
		}
	}

	// Pack hover
	const cardPackSpring1 = spring(1, { stiffness: 0.125, damping: 0.25 })
	const cardPackSpring2 = spring(1, { stiffness: 0.125, damping: 0.25 })
	const cardPackSpring3 = spring(1, { stiffness: 0.125, damping: 0.25 })

	// TASK ----------------------------------- //
	let packRot = 0
	const packRotSpring = spring(0, {
		stiffness: 0.5,
		damping: 0.7
	})

	let rotationDirection = 1
	const MAX_ROTATION = 0.125
	const ROTATION_SPEED = 0.025

	useTask((delta) => {
		// Orbit on cursor move (x axis)
		if (orbitControlsRef) {
			const orbit = orbitControlsRef.object

			const orbitSphere = new THREE.Spherical().setFromVector3(orbit.position)
			orbitSphere.theta = $orbitSpring
			orbit.position.setFromSpherical(orbitSphere)

			orbitControlsRef.update()
		}

		// Pack rotation
		let targetRotation = $packRotSpring + delta * ROTATION_SPEED * rotationDirection

		if (Math.abs(targetRotation) > MAX_ROTATION) {
			rotationDirection *= -1
			targetRotation = Math.sign(targetRotation) * MAX_ROTATION
		}

		packRotSpring.set(targetRotation)
	})
</script>

<!-- DEV -->
<Gizmo size={80}></Gizmo>

<!-- CAMERA -->
<T.PerspectiveCamera
	makeDefault
	position={[1, -2, 12]}
	fov={12}
	on:create={({ ref }) => ref.lookAt(0, 0, 0)}
>
	<OrbitControls
		enableZoom={true}
		enablePan={false}
		enableRotate={false}
		autoRotate={false}
		on:create={({ ref }) => {
			orbitControlsRef = ref
		}}
	/>
</T.PerspectiveCamera>

<!-- LIGHTING -->
<T.DirectionalLight let:ref intensity={1} position={[2, 2, 4]}>
	<!-- <Portal object={scene}>
		<T.DirectionalLightHelper args={[ref, 1, 0xffff00]} bind:ref={directionalLightHelper} />
	</Portal> -->
</T.DirectionalLight>
<T.DirectionalLight intensity={1} position={[12, 2, 12]} color={'#aaaaff'} />

<T.DirectionalLight intensity={0.125} position={[0, 6, 12]} />

<T.AmbientLight intensity={4} color={'#ffaaff'} />

<!-- <Sky elevation={12} rayleigh={0} azimuth={-100} /> -->
<Environment path="/" files="studio.hdr" isBackground={false} format="hdr" />

<!-- MESH -->
<!-- <T.Mesh>
	<T.BoxGeometry args={[1, 1, 1]} />
	<T.MeshPhysicalMaterial
		iridescence={1}
		iridescenceIOR={1}
		iridescenceThicknessRange={[0, 1400]}
		roughness={0.33}
		clearcoat={0.5}
		metalness={0.75}
	/>
</T.Mesh> -->

<!-- <CardPack scale={0.05} position={[0, 0, 0]}></CardPack> -->
<CardPackTex
	on:pointerover={() => cardPackSpring1.set(1.5)}
	on:pointerout={() => cardPackSpring1.set(1)}
	scale={$cardPackSpring1}
	rotation.z={$packRotSpring}
	rotation.y={$packRotSpring * 2}
/>

<!-- <CardPackTex
	position.x={-3}
	rotation.y={-0.5}
	position.z={-1}
	on:pointerover={() => cardPackSpring2.set(1.25)}
	on:pointerout={() => cardPackSpring2.set(1)}
	scale={$cardPackSpring2}
/>

<CardPackTex
	position.x={3}
	rotation.y={0.5}
	position.z={-1}
	on:pointerover={() => cardPackSpring3.set(1.25)}
	on:pointerout={() => cardPackSpring3.set(1)}
	scale={$cardPackSpring3}
/> -->
