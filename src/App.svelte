<!-- Logic -->

<script module>	
	import MainMenu from '$lib/components/MainMenu.svelte';
	import Notifications from '$lib/components/Notifications.svelte';
	
	let CurrentAppScreen = $state(MainMenu);

	export function setAppScreen(componentType) {
		CurrentAppScreen = componentType;
	}
</script>


<!-- Markup -->

<CurrentAppScreen />
<Notifications />


<!-- Styles -->

<style>
	:global {
		#app {
			background-color: rgb(25, 23, 30);

			position: fixed; 
			overflow: none;

			top: 0;
			left: 0;
			width: 100vw;
			height: 100vh;

			align-items: center;
			justify-content: center;

			/* top: 10vh;
			left: 10vw;
			width: 80vw;
			height: 80vh; */

			/*display: flex;
			flex-direction: column;
			> * {
				flex: 1;
			} */
		}

	}
</style>
