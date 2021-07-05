<script>
	import {fly} from 'svelte/transition';

	function hexToRGB(hex,a, pos){
		const [r, g, b] = hex.match(/\w\w/g).map(x => parseInt(x, 16));
		if(pos) {
			pos = pos + '%';
			return `rgba(${r}, ${g}, ${b}, ${a}) ${pos}`;
		}
		return `rgba(${r}, ${g}, ${b}, ${a})`;
	}
	function getRandomColor() {
		return '#' + (Math.random().toString(16) + '000000').substring(2,8).toUpperCase();
	}

	let colorsCounter = 1; // счетчик для определения количества цветов (от 2 до 10 цветов)
	let defaultType = "linear-gradient"; 
	let defaultDirection = "to bottom,";
	let degreesValue = 0; // дефолтное значение градусов для инпута
	let directionFlag = true;
	let conicFlag = false;
	let active = 'linear';

	let conicX = "50";
	let conicY = "50";


	let colorCollection = [
		{id: 1, color: getRandomColor(), colorPosition: "", alpha: 1},
		{id: 2, color: getRandomColor(), colorPosition: "", alpha: 1}
	];

	function editColor(id, colorVal, rangeVal) {
		let elem = colorCollection.filter(el => el.id == id)[0];
		elem.color = colorVal;
		elem.alpha = rangeVal;
		colorCollection = colorCollection;
	};

	function editPosition(id, positionVal) {
		let elem = colorCollection.filter(el => el.id === id);
		elem.colorPosition = positionVal;
		colorCollection = colorCollection;
	};

	function addColor() {
		if(colorsCounter < 8) {
			let newItemColor = {
				id: colorCollection[colorCollection.length-1].id + 1,
				color: getRandomColor(),
				colorPosition: "",
				alpha: 1
			}

			colorsCounter++;
			colorCollection = [...colorCollection, newItemColor];
		}
	};

	function removeColor(id) {
		if(colorsCounter >= 2) {
			colorCollection = colorCollection.filter(el => el.id !== id);
			colorsCounter--;
		}		
	};

	function changeTypeOfGradient(num) {
		if (num === 1) {
			defaultType = "radial-gradient";
			defaultDirection = '';
			directionFlag = false;
			conicFlag = false;
			active = 'radial';
		} else if (num === 2) {
			defaultType = "linear-gradient";
			defaultDirection = '';
			directionFlag = true;
			conicFlag = false;
			active = "linear";
		} else if (num === 3) {
			defaultType = "conic-gradient";
			defaultDirection = 'at 50% 50%,';
			directionFlag = false;
			conicFlag = true;
			active = "conic";
		}
	};
	
	function changeGradientDirection(val, word) {
		defaultDirection = `${val}deg,`;
		if(word) {
			degreesValue = 0;
			defaultDirection = word;
		}
	};

	function changeConicPosition(x,y) {

		defaultDirection = `at ${x}% ${y}%,`;
	}
	

	function getColors(arr) {
		let resultStringColor = arr.map(el => hexToRGB(el.color, el.alpha, el.colorPosition));
		return resultStringColor.join(', ');
	};

	function copyCode() {
		let stringForCopy = document.querySelector('.css_code').innerHTML;
		navigator.clipboard.writeText(stringForCopy);
	};
</script>

<div class="wrapper">
	<div class="gradient" style="background-image: {defaultType}({defaultDirection}{getColors(colorCollection)})"></div>
	<div class="settings">
		<div class="main_settings">
			<div class="gradient_type">
				<h3>Type of gradient</h3>
				<button class:active={active === "radial"} on:click={() => changeTypeOfGradient(1)}>radial</button>
				<button class:active={active === "linear"} on:click={() => changeTypeOfGradient(2)}>linear</button>
				<button class:active={active === "conic"} on:click={() => changeTypeOfGradient(3)}>conic</button>
			</div>

			{#if directionFlag}
				<div class="gradient_direction" in:fly={{y:-20}}>
					<h3>Gradient direction</h3>
					<button class:active={defaultDirection === "to bottom,"} on:click={() => changeGradientDirection(null, "to bottom,")}>bottom</button>
					<button class:active={defaultDirection === "to right,"} on:click={() => changeGradientDirection(null, "to right,")}>right</button>
					<button class:active={defaultDirection === "to left,"} on:click={() => changeGradientDirection(null, "to left,")}>left</button>
					<button class:active={defaultDirection === "to top,"} on:click={() => changeGradientDirection(null, "to top,")}>top</button>
					<div class="manual_inputs">
						<h5>Degree offset</h5>
						<input type="range" min="0" max="360" bind:value={degreesValue} on:input={changeGradientDirection(degreesValue)}>
						<input type="number" bind:value={degreesValue} on:input={changeGradientDirection(degreesValue)}>
					</div>
				</div>
			{/if}

			{#if conicFlag} 
				<div class="conic_pos">
					<h5>Positioning the conic center (x, y)</h5>
					at (0-100%)
					<input type="number" min="0" max="100" on:change={changeConicPosition(conicX, conicY)} bind:value={conicX}>
					<input type="number" min="0" max="100" on:change={changeConicPosition(conicX, conicY)} bind:value={conicY}>
				</div>
				
			{/if}
		</div>
		
		<div class="main_content">
			<div class="controls">
				<button on:click={() => addColor()}>add</button>
			</div>
			<div class="inputs_list">
				<div class="inputs_item">
					<div class="alpha_container">
						<h5>Opacity</h5>
					</div>
					<div class="color_container">
						<h5>Color</h5>
					</div>
					<div class="stop">
						<h5>Stop (%)</h5>
					</div>
					<div class="remove_container"></div>
				</div>
			</div>
	
			<div class="inputs_list">
				{#each colorCollection as item (item.id)}
					<div class="inputs_item">
						<div class="alpha_container">
							<input type="range" min="0" max="1" step="0.01" 
								bind:value={item.alpha}
								on:input={editColor(item.id, item.color, item.alpha)}
								>
							<input type="number" bind:value={item.alpha}>
						</div>
						
						<div class="color_container">
							<label class="color_wrapper" style="background: {item.color}">
								<input type="color" bind:value={item.color} on:input={editColor(item.id, item.color, item.alpha)}>
							</label>
						</div>
						

						<div class="stop">
							<input type="number" bind:value={item.colorPosition} on:input={editPosition(item.id, item.colorPosition)}>
						</div>

						<div class="remove_container">
							{#if colorCollection.length > 2}
								<button on:click={removeColor(item.id)}>&#10006;</button>
							{/if}
						</div>
					</div>
				{/each}
			</div>
		</div>
		<div class="css_code" on:click={() => copyCode()}>
			<h5 style="color: gold">Click to copy</h5>
			background-image: {defaultType}({defaultDirection}{getColors(colorCollection)});
		</div>
	</div>
	
</div>

<style type="scss">
	:global(*){
		margin: 0;
		padding: 0;
		box-sizing: border-box;
		color: #1F2667;
	}

	button {
		background: rgba(0,0,0,0);
		border: 1px solid #d4d4d4;
		border-radius: 6px;
		font-size: 11px;
		padding: 7px 10px;
		text-transform: uppercase;
		flex: 1;
		cursor: pointer;
	}

	input[type=number] {
		width: 70px;
		padding: 10px;
		appearance: none;
		border: 1px solid #d4d4d4;
		border-radius: 6px;
		-moz-appearance: textfield;
	} 

	h3, h5 {
		width: 100%;
	}

	.gradient_type {
		margin-bottom: 40px;
		display: flex;
		flex-wrap: wrap;
		gap: 5px;
	}

	.gradient_direction {
		margin-bottom: 40px;
		display: flex;
		flex-wrap: wrap;
		gap: 5px;
	}

	.manual_inputs {
		display: flex;
		flex-wrap: wrap;
		margin-top: 20px;
	}

	.manual_inputs input[type=range] {
		margin-right: 10px;
		flex: 1;
	}
	
	.active {
		background: #1F2667;
		color: #fff;
		transition: .2s;
	}

	.wrapper {
		height: 100vh;
	}

	.gradient {
		width: 100%;
		height: 300px;
	}

	.settings {
		max-width: 1000px;
		background: #fff;
		box-shadow: 0px 0px 1px 0px rgba(0,0,0, .4);
		margin: 0 auto;
		padding: 30px;
		border-radius: 6px;
		margin-top: -30px;

		display: grid;
		column-gap: 20px;
		grid-template-columns: 1fr 2fr;
		grid-template-areas: 
			"aside main"
			"code code";
	}

	.main_settings {
		grid-area: aside;
	}

	.main_content {
		grid-area: main;
	}

	.css_code {
		grid-area: code;
		background: #131417;
		border-radius: 6px;
		padding: 20px;
		color: #fff;
		font-style: italic;
		cursor: pointer;
		margin-top: 20px;
	}

	input[type=color] {
		opacity: 0;
	}

	.color_wrapper {
		cursor: pointer;
		border-radius: 6px;
		height: 100%;
		width: 100px;
		display: inline-block;
	}

	.inputs_item {
		display: flex;
		gap: 30px;
		margin-bottom: 10px;
	}

	

	.remove_container button {
		position: relative;
		height: 100%;
		width: 40px;
	}


	.alpha_container {
		flex: 3;
		display: flex;
		align-items: center;
		gap: 5px;
	}

	.color_container {
		flex: 1;
	}

	.stop {
		flex: 1;
	}

	.stop input {
		width: 100%;
	}

	.remove_container {
		flex: 1;
	}

	.controls {
		margin-bottom: 20px;
	}

	.conic_pos h5 {
		margin-bottom: 10px;
	}
</style>