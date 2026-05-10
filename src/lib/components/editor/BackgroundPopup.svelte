<!-- Logic -->
<script>
	import { blur } from 'svelte/transition';
    import { pushNotif } from '../Notifications.svelte';

	const { children, onClose, onBackgroundAdd } = $props();

	let currentPanel = $state(0);  // 0-based index
	let currentTab = $state(0);  // 0-based index

	function switchTab(clickedButton, idx=0) {
		currentTab = idx;

		let tabs = Array.from(document.getElementById("tabs").children);
		for (const button of tabs) button.classList.remove("selected");
		clickedButton.classList.add("selected");
	}


	// Panel 2 - 'Add New Background' Logic //

	let solidColor = '#ffffff';
	function setColor(value) {
		solidColor = event.target.value;
	}

	let gradientColors = ['#ffffff', '#ffffff'];
	function setGradientColor(index, value) {
		gradientColors[index] = value;
		gradientColors = [...gradientColors];
	}

	let image = $state('');  // image data as a base64 dataURL
	function setImage(filepath) {
		const reader = new FileReader();
		reader.onload = () => {
			image = reader.result;
		};
		reader.readAsDataURL(filepath);
	}

	// Confirmation

	let scroller = document.getElementById("page-holder");

	function addBackground() {
		let createdBackground = {
			type: null,
			cssDataString: null,
			activationHeight: scroller.scrollTop
		}

		if (currentTab == 0) {  // Solid Color
			createdBackground.type = 'color';
			createdBackground.cssDataString = solidColor;
			console.log(createdBackground);
		}
		else if (currentTab == 1) {  // Gradient
			const getCSSGradient = (colors=gradientColors, direction='90deg', spacing=null) => {
				// TODO: implement custom spacing
				// const step = 100 / (colors.length - 1);
				// const stops = colors.map((c, i) => `${c} ${i * step}%`);
				// return `linear-gradient(${direction}, ${stops.join(", ")})`;

				return `linear-gradient(${direction}, ${colors.join(", ")})`;
			}

			createdBackground.type = 'gradient';
			createdBackground.cssDataString = getCSSGradient(gradientColors, '0deg');
		}
		else if (currentTab == 2) {  // Image
			if (!image) {
				pushNotif("Upload an image before confirming!");
				return;
			}

			createdBackground.type = 'image';
			createdBackground.cssDataString = $state.snapshot(image);
		}

		onBackgroundAdd(createdBackground);
		//currentPanel = 0;
		onClose();
	}
</script>


<!-- Markup -->

<div class="overlay" in:blur, out:blur>
	{#if currentPanel == 0}
		<div class="panel manage">
			<button class="close" onclick={onClose}>&times;</button>

			<h1 style="text-align: center;">Background List</h1>
			<ol id="bg-list">
				{@render children()}

				<li>
					<button title="Add New Background" class="plus" onclick={() => currentPanel = 1}>&plus;</button>
				</li>
			</ol>
		</div>
	{:else if currentPanel == 1}
		<!-- TODO: add back button that sets currentPanel back to 0 -->

		<div class="panel">
			<button class="close" onclick={onClose}>&times;</button>
			
			<div id="tabs">
				<button class="tab selected" onclick={(ev) => switchTab(ev.target, 0)}>Color</button>
				<button class="tab" onclick={(ev) => switchTab(ev.target, 1)}>Gradient</button>
				<button class="tab" onclick={(ev) => switchTab(ev.target, 2)}>Image</button>
			</div>
			
			<hr>
			
			{#if currentTab == 0}
				<!-- Color -->
				<div class="tabcontent">
					<input type="color" value="#ffffff" oninput={ (event) => {setColor(event.target.value)} } />
				</div>
			{:else if currentTab == 1}
				<!-- Gradient -->
				<div class="tabcontent">
					<label>Start Color:
						<input type="color" value="#ffffff" oninput={ (ev) => setGradientColor(0, ev.target.value) } />
					</label>

					<label>End Color:
						<input type="color" value="#ffffff" oninput={ (ev) => setGradientColor(0, ev.target.value) } />
					</label>
				</div>
			{:else if currentTab == 2}
				<!-- Image -->
				<div class="tabcontent" style="display: block;">
					<input type="file" oninput={ (event) => setImage(event.target.files[0]) } />
					<img src={image} height="200" alt="(Preview)" />
				</div>
			{/if}

			<button class="confirm" onclick={addBackground}>Confirm</button>
		</div>
	{/if}
</div>


<!-- Styles -->

<style>
	* {  /* default inline styling */
		vertical-align: middle;
		color: white;
		user-select: none;
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

		&.manage {
			background-color: #aaaaaa;

			ol {
				overflow: auto;
				height: 30vh;
			}

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

		button {
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
	}

</style>