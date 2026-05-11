<!-- logic -->

<script>
	import { setAppScreen } from "/App.svelte";
	import StoryInterface from '$lib/components/StoryViewer.svelte';
</script>


<!-- markup -->

<div id="main">
	<h1>Immersive Writer</h1>

	<section id="button-list" style="margin: 0px;">

		<button onclick={() => setAppScreen(StoryInterface)}>
			New Story
		</button>

		<button>Open File...</button>

	</section>
</div>


<!-- styles -->

<style>

	#main {
		width: 100%;
		height: 100%;
		
		background-color: rgb(32, 32, 32); 
		box-shadow: 0 0 200px rgba(0,0,0,0.9) inset;

		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	h1, button {
		display: block;
		margin: 6% auto 6% auto;
		color: #ffffff;

		font-family: Parchment;
		font-size: 6rem;
		text-align: center;

		user-select: none;
	}

	button {
		margin: 12% auto 12% auto;
		padding: 4px;
		
		background-color: rgb(187, 199, 209);
		border-top-right-radius: 12px;
		border-bottom-left-radius: 12px;

		outline-width: 5px;
		outline-color: black;

		font-size: 2rem;
		font-family: Maiandra;
		text-shadow: 0px 0px 3px black;
		text-align: center;

		transition: 180ms;
	}

	button:hover {
		box-shadow: 0 0 4px 3px rgba(218, 218, 218, 0.6);
		transform: scale(1.03);
		transition: 180ms;
	}

</style>
