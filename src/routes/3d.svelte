<script>
	import typefaceData from '@compai/font-recursive/data/typefaces/normal-400.json';
	import { onMount } from 'svelte';
	import * as SC from 'svelte-cubed';
	import * as THREE from 'three';
	import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry.js';
	import { FontLoader } from 'three/examples/jsm/loaders/FontLoader.js';
	import { STLLoader } from 'three/examples/jsm/loaders/STLLoader.js';

	let width = 1;
	let height = 1;
	let depth = 1;

	let spin = 0;
	const fontLoader = new FontLoader();
	let font = fontLoader.parse(typefaceData);

	SC.onFrame(() => {
		spin += 0.01;
	});
	let geometry = null;
	const material = new THREE.MeshPhysicalMaterial({
		color: 0xbbffcc,
		// envMap: envTexture,
		metalness: 0.23,
		roughness: 0.2,
		opacity: 1.0,
		transparent: true,
		transmission: 0.99,
		clearcoat: 1.0,
		clearcoatRoughness: 0.25
	});

	onMount(() => {
		const loader = new STLLoader();
		loader.load('/model.stl', function (g) {
			geometry = g;
		});
	});

	$: createText = (/** @type {string} */ str) => {
		if (!font) return;
		const textGeometry = new TextGeometry(str, {
			font,
			size: 5,
			height: 0.25,
			curveSegments: 12
		});
		textGeometry.center();
		return textGeometry;
	};
</script>

<SC.Canvas
	antialias
	background={new THREE.Color('papayawhip')}
	fog={new THREE.FogExp2('papayawhip', 0.1)}
	shadows
>
	<SC.Group position={[0, -height / 2, 0]}>
		<SC.Mesh
			geometry={new THREE.PlaneGeometry(50, 50)}
			material={new THREE.MeshStandardMaterial({ color: 'burlywood' })}
			rotation={[-Math.PI / 2, 0, 0]}
			receiveShadow
		/>
		<SC.Primitive
			object={new THREE.GridHelper(50, 50, 'papayawhip', 'papayawhip')}
			position={[0, 0.001, 0]}
		/>
	</SC.Group>

	{#if geometry}
		<SC.Group rotation={[0, spin, 0]} position={[0, 0.6, 0]}>
			<SC.Mesh
				{geometry}
				{material}
				scale={[0.005 * width, 0.005 * height, 0.005 * depth]}
				rotation={[-Math.PI / 2, 0, Math.PI / 2]}
				castShadow
			/>
		</SC.Group>
	{:else}
		<SC.Group>
			<SC.Mesh
				geometry={new THREE.BoxGeometry()}
				material={new THREE.MeshStandardMaterial({ color: 0xff3e00 })}
				scale={[width, height, depth]}
				rotation={[0, spin, 0]}
				castShadow
			/>
			<SC.Mesh
				geometry={createText('loading...')}
				{material}
				position={[0, 0, 1]}
				scale={[0.1, 0.1, 0.1]}
				rotation={[0, 0, 0]}
				castShadow
			/>
		</SC.Group>
	{/if}

	<SC.PerspectiveCamera position={[1, 1, 3]} />
	<SC.OrbitControls enableZoom={false} maxPolarAngle={Math.PI * 0.51} />
	<SC.AmbientLight intensity={0.6} />
	<SC.DirectionalLight intensity={0.6} position={[-2, 3, 2]} shadow={{ mapSize: [2048, 2048] }} />
</SC.Canvas>

<div class="controls">
	<label><input type="range" bind:value={width} min={0.1} max={3} step={0.1} /> width</label>
	<label><input type="range" bind:value={height} min={0.1} max={3} step={0.1} /> height</label>
	<label><input type="range" bind:value={depth} min={0.1} max={3} step={0.1} /> depth</label>
</div>

<style>
	.controls {
		position: absolute;
		left: 1em;
		top: 1em;
	}

	label {
		display: flex;
		width: 60px;
		gap: 0.5em;
		align-items: center;
	}

	input {
		width: 80px;
		margin: 0;
	}
</style>
