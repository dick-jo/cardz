<script>
	import SceneTwo from '$lib/SceneTwo.svelte'
	import { Canvas } from '@threlte/core'
</script>

<!-- MARKUP -->
<main>
	<Canvas>
		<!-- <SceneOne /> -->
		<SceneTwo />
	</Canvas>
</main>

<!-- CSS -->
<style>
	:global(*) {
		padding: 0;
		margin: 0;
		box-sizing: border-box;
	}

	:global(html, body) {
		width: 100%;
		height: 100%;
	}

	:global(body) {
		padding: 40px;
	}

	main {
		width: 100%;
		height: 100%;
		/* background-color: #ffa0be; */
	}
</style>
