<!-- Logic -->

<script>
	import EditorOverlay, { setEditorPage } from './editor/EditorOverlay.svelte';
	import { onMount } from 'svelte';
	
	// Reactive Story View //

	let storyData = $state({
		dynamicBgList: [],  // Array of dynamic background data objects
		propList: [],  // Array of prop data objects

		scrollHeight: 0  // Current story reading progress
	});

	onMount(() => {
		let pageHolder = document.getElementById("page-holder");

		const onScroll = () => {
			storyData.scrollHeight = pageHolder.scrollTop;
		};

		pageHolder.addEventListener('scroll', onScroll);
		return () => pageHolder.removeEventListener('scroll', onScroll);
	});

	/** Used for Dynamically Blended Backgrounds
	 * @param showHeight - the height at which the background becomes fully opaque.
	 * @param graduality - the 'leeway' or 'smoothness' of the opacity transition in pixels.
	 * @returns {number} An opacity value from 0-1 based on how close the current scroll position is to showHeight.
	 */
	function calculateBgOpacity(showHeight=0, graduality=200) {
		let raw = (graduality - (showHeight - storyData.scrollHeight)) / graduality;
		return Math.max(0, Math.min(1, raw));
	}

	// Editor Overlay //

	const InterfaceModes = Object.freeze({
		READ: 0,
		EDIT: 1
	});

	let currentMode = $state(InterfaceModes.READ);

	function setMode(nextMode) {
		if (!nextMode) nextMode = (currentMode == InterfaceModes.READ) ? InterfaceModes.EDIT : InterfaceModes.READ;

		if (nextMode == InterfaceModes.EDIT) {
			setEditorPage(document.querySelector(".page"));  // edit the closest page to the center of the screen (TODO - below is placeholder code)
		}
		else {
			setEditorPage(null);  // stop editing
		}

		currentMode = nextMode;
	}

	function setStoryData(obj) {
		storyData = obj;
	}

</script>


<!-- Markup -->

<div class="layer background">
	<!-- Dynamically Blended Backgrounds -->
	{#each storyData.dynamicBgList as dynamicBg}
		{#if dynamicBg.type === 'image'}
			<div class="dynamic-bg" style="
				opacity: {calculateBgOpacity(dynamicBg.activationHeight)};
				background-image: url({dynamicBg.cssDataString});
			"></div>

		{:else if dynamicBg.type === 'gradient'}
			<div class="dynamic-bg" style="
				opacity: {calculateBgOpacity(dynamicBg.activationHeight)};
				background-image: {dynamicBg.cssDataString};
			"></div>

		{:else if dynamicBg.type === 'color'}
			<div class="dynamic-bg" style="
				opacity: {calculateBgOpacity(dynamicBg.activationHeight)};
				background-color: {dynamicBg.cssDataString};
			"></div>

		{/if}
	{/each}
</div>

<div class="layer" id="page-holder">

	{#each storyData.propList as prop}
		<img 
			class = "prop" 
			src = {prop.cssDataString} 
			style = "top: {prop.positionY}px; left: {prop.positionX}px; transform: rotate({prop.rotation}deg);" 
			alt = "user-uploaded story prop"
		>
	{/each}

	<div class="page endless" style="color: white; font-size: 2.8vmin;">
		<span style="font-weight:bold; font-size: 3.2vmin;">Welcome!</span>
	</div>
</div>

<div class="layer foreground">
	<!-- elements will be inserted here dynamically -->
</div>

{#if currentMode == InterfaceModes.EDIT}
	<EditorOverlay storyData={storyData} setStoryData={setStoryData} />
{:else}
	<button id="corner-button" onclick={() => setMode()}>O</button>
{/if}


<!-- Styles -->

<style>
	* {
		margin: auto;
	}

	.layer {
		position: absolute;
		inset: 5vmin 0 0 0;

		width: 100%;
		height: 100%;

		align-items: center;
		justify-content: center;

		overflow: auto;

		&.background {
			z-index: -1;
			pointer-events: none;
		}

		&.foreground {
			z-index: 64;
			pointer-events: none;
		}
	}

	.dynamic-bg {
		position: fixed;
		inset: 0;
		overflow: hidden;

		background-size: cover;
		background-position: center;
		/* transition: opacity 0.2s linear; */
	}

	.prop {
		position: absolute;
		top: 0;
		left: 0;
	}

	.page {
		width: 85vmin;
		aspect-ratio: calc(8.5/11);

		overflow: hidden;

		margin-top: 45px;
		margin-bottom: 25px;
		padding: 5vmin;

		border: solid #ffffff2a 2px;
		
		&.endless {
			aspect-ratio: auto;  /* unlocks height from width */
		}
	}
</style>