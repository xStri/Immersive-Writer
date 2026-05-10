<!-- Logic -->

<script module>
	import { Editor } from '@tiptap/core'

	import StarterKit from '@tiptap/starter-kit'
	import Underline from '@tiptap/extension-underline'
	import { TextStyle } from '@tiptap/extension-text-style'
	import Color from '@tiptap/extension-color'
	import FontFamily from '@tiptap/extension-font-family'

	import { FontSize } from '$lib/tiptap/extensions/font_size.js'

	let editor = new Editor({
		element: null,
		extensions: [
			StarterKit,
			Underline,
			TextStyle,
			Color,
			FontFamily,
			FontSize
		]
	});


	let currentPage;

    export function setEditorPage(page=null) {
		if (page) {
			editor.commands.setContent(page.innerHTML);
			page.innerHTML = "";
			editor.mount(page);
		}
		else {
			currentPage.innerHTML = editor.getHTML();
			editor.unmount();
		}
		
		currentPage = page;
	}

	export function getEditorPage() {
		return currentPage;
	}

	// Toolbar Functions //

	let currentFont = $state('Arial');
	let currentColor = $state('#000000');
	let currentSize = $state(12);

	function setFont(event) {
		let inputElement = event.target;

		currentFont = inputElement.value;
		editor.chain().focus().setFontFamily(currentFont).run();
	}

	function setColor(event) {
		let inputElement = event.target;

		currentColor = inputElement.value;
		editor.chain().focus().setColor(currentColor).run();
	}

	function setSize(event) {
		let inputElement = event.target;

		currentSize = inputElement.value;
		editor.chain().focus().setFontSize(inputElement.value*0.1 + 'vmin').run()
	}
</script>


<script>
	import BackgroundPopup from './BackgroundPopup.svelte';
    import DecorationPopup from './DecorationPopup.svelte';

	import * as Menubar from '$lib/components/ui/menubar/index.js';
    import { pushNotif } from '../Notifications.svelte';
	// import '/src/app.css';  // commented out because this breaks the ui of my whole app lol

    const { storyData, setStoryData } = $props();
	let pageHolder = document.getElementById("page-holder");

	// Story Saving //

	function saveStoryAs() {
		setEditorPage(null);

		let storyObject = {
			pageHTML: [],  // array containing innerHTML of each page
			storyData: $state.snapshot(storyData)
		}
		
		for (const page of pageHolder.children) storyObject.pageHTML.push(page.innerHTML);

		console.log("Saving story data to file: ", storyObject);

		// serialize & save
		let storyFileData = JSON.stringify(storyObject);
		const blob = new Blob([storyFileData], {
			type: 'application/json',
		});
		
		const url = URL.createObjectURL(blob);
		const a = document.createElement('a');
		a.href = url;
		a.download = `testStory.json`;
		a.click();

		URL.revokeObjectURL(url);

		setEditorPage(document.querySelector(".page"));
	}

	// Story Loading //

	let fileInput;  // bound

	function loadStoryFrom() {

		function onFileChosen(event) {
			fileInput.removeEventListener('change', onFileChosen);

			setEditorPage(null);

			const file = event.target.files[0];

			if (file) {
				const reader = new FileReader();

				reader.onload = () => {
					let storyFileData = reader.result;

					let storyObject = JSON.parse(storyFileData);

					let pIdx = 0;
					for (const pageHTML of storyObject.pageHTML) {
						if (!pageHTML) continue;
						
						let page;
						if (pIdx >= pageHolder.children.length) {
							// Create new page
							page = null;  // TODO!!
						}
						else {
							// Use existing page
							page = pageHolder.children[pIdx];
						}

						page.innerHTML = pageHTML;
						pIdx++;
					}

					for (const [key, value] of Object.entries(storyObject.storyData)) {
						if (key == "scrollHeight") continue;
						storyData[key] = value;
					}
					
					setEditorPage(document.querySelector(".page"));
				};

				reader.readAsText(file);
			}
			else {
				pushNotif("Unable to read file - ", file);
				setEditorPage(document.querySelector(".page"));
			}
		}

		fileInput.addEventListener('change', onFileChosen);
		fileInput.click();
	}


	// Modal Visibility //

	let showBackgroundPopup = $state(false);
	let showDecorationPopup = $state(false);

	function doPropPlacement(propData) { 

		let propIdx = storyData.propList.push(propData)-1;
		propData = storyData.propList[propIdx];

		function onMove(event) {
			// update prop position
			propData.positionX = event.clientX;
			propData.positionY = event.clientY+storyData.scrollHeight;
		}

		function onConfirm() {
			document.removeEventListener("mousemove", onMove);
			document.removeEventListener("mousedown", onConfirm);
		}

		document.addEventListener("mousemove", onMove);
		document.addEventListener("mousedown", onConfirm);
	}

</script>


<!-- Markup -->

<input type="file" style="display: none;" bind:this={fileInput} />

<Menubar.Root style="background-color: #9fb4d8;">
	<Menubar.Menu>
		<Menubar.Trigger> File </Menubar.Trigger>
		<Menubar.Content style="background-color: white; user-select: none;">
			<Menubar.Item> New </Menubar.Item>
			<Menubar.Item onclick={loadStoryFrom}> Open </Menubar.Item>
			<Menubar.Item onclick={saveStoryAs}> Save As... </Menubar.Item>
		</Menubar.Content>
	</Menubar.Menu>
	<Menubar.Menu>
		<Menubar.Trigger> Edit </Menubar.Trigger>
		<Menubar.Content style="background-color: white; user-select: none;">
			<Menubar.Item> Undo <Menubar.Shortcut>Ctrl+Z</Menubar.Shortcut> </Menubar.Item>
			<Menubar.Item> Redo <Menubar.Shortcut>^Y</Menubar.Shortcut> </Menubar.Item>
		</Menubar.Content>
	</Menubar.Menu>
	<Menubar.Menu>
		<Menubar.Trigger> View </Menubar.Trigger>
		<Menubar.Content style="background-color: white; user-select: none;">
			<Menubar.Item> Reading Mode </Menubar.Item>
		</Menubar.Content>
	</Menubar.Menu>
</Menubar.Root>

<div id="toolbar">
    <!-- TODO HERE: dropdown that allows the user to select a Text Styling Preset -->

    <!-- Bold, Italics, Underline -->
    <button class="square"
        onclick={() => editor.chain().focus().toggleBold().run()}
        class:active={editor?.isActive('bold')}
    >B</button>

    <button class="square"
        onclick={() => editor.chain().focus().toggleItalic().run()}
        class:active={editor?.isActive('italic')}
    >I</button>
    
    <button class="square"
        onclick={() => editor.chain().focus().toggleUnderline().run()}
        class:active={editor?.isActive('underline')}
    >U</button>
    
    <input type="color" bind:value={currentColor} oninput={setColor} />

    <select bind:value={currentFont} onchange={setFont}>
        <option value="Arial">Arial</option>
        <option value="Times New Roman">Times New Roman</option>
        <option value="Courier New">Courier New</option>
    </select>

    <label for="font-size">Font Size:</label>
    <input type="number" id="font-size" name="font-size" min="1" max="200" 
        bind:value={currentSize} oninput={setSize}
    /> 

    <!-- TODO: Other Text Effects -->
    <!-- TODO: Horizontal Text Align picker (left, center, right)-->
    <p>|</p>
    <button onclick={() => showBackgroundPopup = true}>Background</button>
    <p>|</p>
    <button onclick={() => showDecorationPopup = true}>Decoration</button>
</div>

{#if showBackgroundPopup}
    <!-- TODO: this could be more organized maybe -->
	<BackgroundPopup onClose={() => showBackgroundPopup = false} onBackgroundAdd={(newBg) => storyData.dynamicBgList.push(newBg)}>
		{#each storyData.dynamicBgList as dynamicBg, idx}
			<li style="display: flex; height: 10vh; aspect-ratio: 1.0;">
				{#if dynamicBg.type === 'image'}
					<img src="{dynamicBg.cssDataString}" alt="user-uploaded background {idx+1}" style="object-fit: contain;">
				{:else if dynamicBg.type === 'gradient'}
					<div style="height: 100%; background-image: {dynamicBg.cssDataString};"></div>
				{:else if dynamicBg.type === 'color'}
					<div style="height: 100%; background-color: {dynamicBg.cssDataString};"></div>
				{/if}

				<button title="Delete Background {idx+1}" class="minus" onclick={() => storyData.dynamicBgList.splice(idx, 1)}>&minus;</button>
			</li>
		{/each}
	</BackgroundPopup>
{/if}

{#if showDecorationPopup}
	<DecorationPopup onClose={() => showDecorationPopup = false} onPropAdd={doPropPlacement}>
		{#each storyData.propList as prop, idx}
			<li style="display: flex; height: 10vh; aspect-ratio: 1.0;">
				{#if prop.type === 'image'}
					<img src="{prop.cssDataString}" alt="user-uploaded background {idx+1}" style="object-fit: contain;">
				{/if}

				<button title="Delete Prop {idx+1}" class="minus" onclick={() => storyData.dynamicBgList.splice(idx, 1)}>&minus;</button>
			</li>
		{/each}
	</DecorationPopup>
{/if}


<!-- Styles -->

<style>

	#toolbar {
		z-index: 128;

		background-color: #cecece;

		position: sticky;
		width: 100%;
		height: 3vmin;

		/* top: 2vmin; */
		margin-bottom: 16px;

		display: flex;
		justify-content: flex-start;
		align-items: center;
		gap: 8px;

		* {
			margin: 0px;
			user-select: none;
			height: 75%;
		}

		input, button, select {
			&:hover {
				transform: scale(1.1);
				transition: 115ms;
			}

			&:active {
				transform: scale(0.9);
				transition: 115ms;
			}
		}

		button {

			background-color: #a1a1a1;
			color: #ffffff;

			font-family: Arial;
			font-weight: bold;

			border-radius: 7px;
			border-width: 0;

			&:hover {
				background-color: #b7c2bf;
			}

			&:active {
				transform: scale(0.8);
			}
		}
	}	
	
	.square {
		aspect-ratio: 1.0;
	}

</style>
