<!-- Logic -->
<script>
	import { blur } from 'svelte/transition';
    import { pushNotif } from '../Notifications.svelte';

	const { children, onClose, onPropAdd } = $props();

	let currentPanel = $state(0);  // 0-based index
	let currentTab = $state(0);  // 0-based index

	function switchTab(clickedButton, idx=0) {
		currentTab = idx;

		let tabs = Array.from(document.getElementById("tabs").children);
		for (const button of tabs) button.classList.remove("selected");
		clickedButton.classList.add("selected");
	}


	// Panel 2 - 'Add New Prop' Logic //

	let image = $state('');  // image data as a base64 dataURL
	function setImage(filepath) {
		const reader = new FileReader();
		reader.onload = () => {
			image = reader.result;
		};
		reader.readAsDataURL(filepath);
	}

	// Confirmation

	let scroller = document.getElementById("middleground");

	function addProp() {
		if (!image) {
			pushNotif("Upload an image before confirming!");
			return;
		}

		let createdProp = {
			type: 'image',
			cssDataString: image,

			// determined by onPropAdd (doPropPlacement)
			positionX: 0,
			positionY: 0,
			rotation: 0
		}

		onPropAdd(createdProp);
		onClose();
	}
</script>


<!-- Markup -->

<div class="overlay" in:blur, out:blur>
	<div class="panel gray">
		<button class="close" onclick={onClose}>&times;</button>
		
		<div id="tabs">
			<button class="tab selected" onclick={(ev) => switchTab(ev.target, 0)}>Particles</button>
			<button class="tab" onclick={(ev) => switchTab(ev.target, 1)}>Decoration</button>
		</div>
		
		<hr>
		
		{#if currentTab == 0}
			<!-- Particles -->
			<div class="tabcontent">
				<label>Start Color:
					<input type="color" value="#ffffff" oninput={ (ev) => setGradientColor(0, ev.target.value) } />
				</label>

				<label>End Color:
					<input type="color" value="#ffffff" oninput={ (ev) => setGradientColor(0, ev.target.value) } />
				</label>
			</div>
		{:else if currentTab == 1}
			<!-- Props -->
			<div class="tabcontent" style="display: block;">
				<h1 style="text-align: center;">Prop List</h1>
				<ol id="bg-list">
					{@render children()}

					<li>
						<button title="Add New Prop" class="plus" onclick={() => currentPanel = 1}>&plus;</button>
					</li>
				</ol>
			</div>
		{/if}


	</div>
</div>

{#if currentPanel == 1}
	<!-- Image Upload Panel -->
	<div class="overlay">
		<div class="panel">
			<input type="file" oninput={ (event) => setImage(event.target.files[0]) } />
			<button class="confirm" onclick={addProp}>Confirm</button>
		</div>
	</div>
{/if}

<!-- Styles -->

<style>
	* {  /* default inline styling */
		vertical-align: middle;
		color: white;
		user-select: none;
		font-family: 'Times New Roman', Times, serif;
	}

	.overlay {
		z-index: 200;

		position: fixed; /* Stay in place */
		width: 100%;
		height: 100%;

		overflow: auto;

		background-color: rgba(0,0,0,0.5);

		box-shadow: 0 0 200px rgba(0,0,0,0.9) inset;

		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.panel {
		background-color: #6a617e;
		width: 69%;
		padding: 20px;
		
		border-radius: 9px;

		hr {
			color: #635084;
			width: 90%;
			border-width: 4px;
			border-style: dashed;
		}

		&.gray {
			background-color: #aaaaaa;

			.close {
				color: #898ea8;

				&:hover, &:focus {
					color: #c6dcf0;
					text-decoration: none;
					cursor: pointer;
				}
			}
		}
	}

	ol {
		overflow: auto;
		height: 30vh;
	}
	
	.close {
		background-color: rgba(0, 0, 0, 0);
		border-color: rgba(0, 0, 0, 0);

		color: #aaaaaa;
		float: right;
		font-size: 28px;
		font-weight: bold;

		&:hover, &:focus {
			color: #ffffff;
			text-decoration: none;
			cursor: pointer;
		}

		user-select: none;
		transition: 215ms;
	}

	#tabs, .tabcontent {
		padding-top: 10px;
		
		display: flex;
		justify-content: center;
		align-items: flex-start;
		
		gap: 16px;

	}

	#tabs > button {
		background-color: rgba(0, 0, 0, 0);
		/* background-color: rgba(235, 235, 235, 0.156); */
		font-family: 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
		font-size: 1.5em;
		color: #ffffff;

		border-width: 0px;
		border-radius: 18px;

		width: 6em;
		height: 100%;
		margin: 15px;

		&:hover {
			/* background-color: rgba(255, 255, 255, 0.2); */
			transform: scale(1.1);
		}
		&.selected {
			text-decoration: underline;
		}

		transition: 215ms;
	}

	#bg-list {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;

		list-style-type: none;

		li {
			height: 10vh;
			width: auto;
			margin: auto;
		}

		li > button {
			height: 100%;
			aspect-ratio: 1.0;
		}
	}

	.plus {
		background-color: #bcc0c0;
		border: dashed #8b8b8b 8px;
		border-radius: 18px;

		font-size: 4em;
		font-weight: bold;
		color: gray;
		text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);

		&:hover {
			box-shadow: 8px 8px 4px rgba(128, 118, 216, 0.5);
			transform: scale(1.1);
		}

		transition: 215ms;
	}

</style>