<script lang="ts">
	let knownLocation: boolean;
	let demoDialog: HTMLDialogElement;
	let weatherData: {
		hourly: { apparent_temperature: any; precipitation: any; windspeed_180m: any };
		daily: { sunrise: any[]; sunset: any[] };
	};
	let temp: number[];
	let sunrise: Date;
	let sunset: Date;
	let sunriseHour: number;
	let sunsetHour: number;
	let precipitation: number[];
	let windSpeed: number[];

	async function getWeatherData(lat: number, lon: number) {
		const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&hourly=apparent_temperature,precipitation_probability,precipitation,cloudcover,windspeed_180m&daily=sunrise,sunset&forecast_days=1&timezone=auto`;
		const response = await fetch(url, {});
		if (!response.ok) {
			throw new Error(`Failed to fetch weather data: ${response.status}`);
		}
		setTimeout(async () => {
			weatherData = await response.json();
			temp = weatherData.hourly.apparent_temperature;
			sunrise = weatherData.daily.sunrise[0];
			sunset = weatherData.daily.sunset[0];
			sunriseHour = new Date(sunrise).getHours();
			sunsetHour = new Date(sunset).getHours();
			precipitation = weatherData.hourly.precipitation;
			windSpeed = weatherData.hourly.windspeed_180m;
			console.log(weatherData);
		}, 3000);
	}

	if (typeof navigator !== 'undefined') {
		navigator.geolocation.getCurrentPosition(
			async (position) => {
				knownLocation = true;
				const lat = position.coords.latitude;
				const lon = position.coords.longitude;
				try {
					await getWeatherData(lat, lon);
				} catch (error) {
					console.error(error);
				}
			},
			(error) => {
				console.error(error);
			}
		);
	}
	let minTemp: number;
	let minRain: number;
	let maxWind: number;

	// check if the minTemp exists in localStorage if not add it, if it does then use it
	if (typeof localStorage !== 'undefined') {
		if (localStorage.getItem('minTemp')) {
			minTemp = Number(localStorage.getItem('minTemp'));
		} else {
			minTemp = 5;
			localStorage.setItem('minTemp', minTemp.toString());
		}
		if (localStorage.getItem('minRain')) {
			minRain = Number(localStorage.getItem('minRain'));
		} else {
			minRain = 0.5;
			localStorage.setItem('minRain', minRain.toString());
		}
		if (localStorage.getItem('maxWind')) {
			maxWind = Number(localStorage.getItem('maxWind'));
		} else {
			maxWind = 60;
			localStorage.setItem('maxWind', maxWind.toString());
		}
	}

	function isGoodWeather() {
		// check if the weather is dry and warm between sunrise and sunset
		for (let i = sunriseHour + 2; i <= sunsetHour; i++) {
			if (temp[i] < minTemp) {
				return false;
			}
			if (precipitation[i] > minRain) {
				return false;
			}
			if (windSpeed[i] > maxWind) {
				return false;
			}
		}
		return true;
	}
</script>

<svelte:head>
	<title>Whirligig Weather</title>
</svelte:head>
<main>
	<header>
		<h1>Whirligig weather tomorrow?</h1>
		<menu on:click={() => demoDialog.showModal()}>
			<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24">
				<path
					fill="currentColor"
					d="M6.17 18a3.001 3.001 0 0 1 5.66 0H22v2H11.83a3.001 3.001 0 0 1-5.66 0H2v-2h4.17zm6-7a3.001 3.001 0 0 1 5.66 0H22v2h-4.17a3.001 3.001 0 0 1-5.66 0H2v-2h10.17zm-6-7a3.001 3.001 0 0 1 5.66 0H22v2H11.83a3.001 3.001 0 0 1-5.66 0H2V4h4.17zM9 6a1 1 0 1 0 0-2 1 1 0 0 0 0 2zm6 7a1 1 0 1 0 0-2 1 1 0 0 0 0 2zm-6 7a1 1 0 1 0 0-2 1 1 0 0 0 0 2z"
				/>
			</svg>
		</menu>
		<dialog bind:this={demoDialog}>
			<div>
				<label for="minTemp">
					Minimum temperature • {minTemp}°C
					<input
						type="range"
						name="minTemp"
						id="minTemp"
						min="-10"
						max="30"
						step="1"
						bind:value={minTemp}
						on:change={() => {
							localStorage.setItem('minTemp', minTemp.toString());
						}}
					/>
				</label>

				<label for="minRain">
					Minimum rain • {minRain}mm
					<input
						type="range"
						name="minRain"
						id="minRain"
						min="0"
						max="5"
						step="0.1"
						bind:value={minRain}
						on:change={() => {
							localStorage.setItem('minRain', minRain.toString());
						}}
					/>
				</label>

				<label for="maxWind">
					Maximum wind • {maxWind}km/h
					<input
						type="range"
						name="maxWind"
						id="maxWind"
						min="0"
						max="100"
						step="5"
						bind:value={maxWind}
						on:change={() => {
							localStorage.setItem('maxWind', maxWind.toString());
						}}
					/>
				</label>

				<button on:click={() => demoDialog.close()}>Close + Save</button>
			</div>
		</dialog>
	</header>
	{#key minTemp + minRain + maxWind}
		<div
			class="wrapper
				{weatherData && isGoodWeather() ? 'good' : ''} 
				{weatherData && !isGoodWeather() ? 'bad' : ''}"
		>
			{#if knownLocation}
				{#if weatherData}
					<div class="message">
						{#if isGoodWeather()}
							<p>Yup!</p>
						{:else}
							<p>Nope.</p>
						{/if}
					</div>
				{:else}
					<div class="loading"><p>Checking...</p></div>
				{/if}
			{:else}
				<div class="location">
					<p>Where are you?</p>
				</div>
			{/if}
		</div>
	{/key}
</main>

<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link
	href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@700&display=swap"
	rel="stylesheet"
/>

<style>
	:root {
		--good-line: greenyellow;
		--good-background: darkgreen;
		--bad-background: hsl(0, 100%, 36%);
		--bad-stripe: hsla(0deg, 100%, 0%, 0.875);
		--bad-stripe-highlight: hsla(0deg, 100%, 0%, 1);
		--default-background: hsl(180, 100%, 25.1%);
		--dialog-background: hsla(180, 100%, 17.5%, 0.9);
		--loading-stripe: hsl(180, 100%, 17.5%);
		--location-stripe: hsl(190, 75%, 50%);
		--text: white;
		--text-shadow: 0 0.25em 0 black;
		font-family: 'Space Grotesk', sans-serif;
		line-height: 0.9;
		color: var(--text);
		font-weight: 700;
		text-shadow: var(--text-shadow);
	}

	main {
		display: grid;
		grid-template-rows: auto 1fr;
		grid-template-columns: 100%;
	}
	header {
		grid-row: 1/2;
		grid-column: 1/-1;
		z-index: 2;
		--webkit-backdrop-filter: blur(2rem);
		backdrop-filter: blur(2rem);
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 2rem;
	}
	header h1 {
		font-size: 3rem;
		margin: 0;
	}
	menu {
		margin: 0;
		padding: 0;
		cursor: pointer;
	}
	menu svg {
		height: 2rem;
		width: 2rem;
		fill: white;
		filter: drop-shadow(var(--text-shadow));
	}
	dialog {
		text-shadow: 0 0 0 transparent;
		padding: 0;
		border: none;
		width: 80vw;
		background-color: hsla(0, 0%, 100%, 0.9);
	}
	dialog::backdrop {
		backdrop-filter: blur(2rem);
	}
	dialog div {
		display: flex;
		flex-direction: column;
		gap: 2rem;
		padding: 2rem;
		border: 0;
	}
	dialog label {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}
	dialog button {
		font-family: 'Space Grotesk', sans-serif;
		appearance: none;
		outline: none;
		background-color: transparent;
		border: 0.25rem solid black;
		box-shadow: var(--text-shadow);
		font-size: clamp(1rem, 2vw, 30rem);
		white-space: nowrap;
		padding: 1rem;
		font-weight: 700;
		transition: all 150ms linear;
		cursor: pointer;
		text-transform: uppercase;
	}
	dialog button:hover {
		background-color: black;
		color: white;
		box-shadow: none;
		transform: translateY(0.25rem);
	}
	.wrapper {
		grid-row: 1/3;
		grid-column: 1/-1;
		display: flex;
		flex-direction: column;
		height: 100vh;
		background-color: var(--default-background);
		transition: all 450ms linear;
	}

	.message {
		flex: 1;
		display: grid;
		place-items: center;
		grid-template: 1fr / 1fr;
		font-size: clamp(7rem, 15vw, 30rem);
		opacity: 1;
		letter-spacing: -0.125ch;
		text-align: center;
	}
	.message p {
		grid-column: 1/-1;
		grid-row: 1/-1;
		padding-inline: 2rem;
	}
	.bad {
		background-color: var(--bad-background);
	}
	.bad .message {
		background-image: repeating-linear-gradient(
				45deg,
				var(--bad-stripe-highlight) 0vmin,
				var(--bad-stripe-highlight) 1vmin,
				var(--bad-stripe) 1vmin,
				var(--bad-stripe) 5vmin,
				transparent 5vmin,
				transparent 10vmin
			),
			repeating-linear-gradient(
				-45deg,
				var(--bad-stripe-highlight) 0vmin,
				var(--bad-stripe-highlight) 1vmin,
				var(--bad-stripe) 1vmin,
				var(--bad-stripe) 5vmin,
				transparent 5vmin,
				transparent 10vmin
			);
		background-position: center center;
	}
	.good {
		background-color: var(--good-background);
	}
	.good .message {
		z-index: 1;
		overflow: hidden;
	}
	.good .message::before {
		content: '';
		z-index: -1;
		height: 0;
		width: 0;
		aspect-ratio: 1/1;
		grid-column: 1/-1;
		grid-row: 1/-1;
		box-shadow: 0 0 0 1vmax var(--good-line), 0 0 0 5vmax var(--good-background),
			0 0 0 6vmax var(--good-line), 0 0 0 10vmax var(--good-background),
			0 0 0 11vmax var(--good-line), 0 0 0 15vmax var(--good-background),
			0 0 0 16vmax var(--good-line), 0 0 0 20vmax var(--good-background),
			0 0 0 21vmax var(--good-line), 0 0 0 25vmax var(--good-background),
			0 0 0 26vmax var(--good-line), 0 0 0 30vmax var(--good-background),
			0 0 0 31vmax var(--good-line), 0 0 0 35vmax var(--good-background),
			0 0 0 36vmax var(--good-line), 0 0 0 40vmax var(--good-background),
			0 0 0 41vmax var(--good-line), 0 0 0 45vmax var(--good-background),
			0 0 0 46vmax var(--good-line), 0 0 0 50vmax var(--good-background),
			0 0 0 51vmax var(--good-line), 0 0 0 55vmax var(--good-background),
			0 0 0 56vmax var(--good-line), 0 0 0 60vmax var(--good-background),
			0 0 0 61vmax var(--good-line), 0 0 0 65vmax var(--good-background),
			0 0 0 66vmax var(--good-line), 0 0 0 70vmax var(--good-background),
			0 0 0 71vmax var(--good-line), 0 0 0 75vmax var(--good-background),
			0 0 0 76vmax var(--good-line), 0 0 0 80vmax var(--good-background),
			0 0 0 81vmax var(--good-line), 0 0 0 85vmax var(--good-background),
			0 0 0 86vmax var(--good-line), 0 0 0 90vmax var(--good-background),
			0 0 0 91vmax var(--good-line), 0 0 0 95vmax var(--good-background),
			0 0 0 96vmax var(--good-line), 0 0 0 100vmax var(--good-background);
		animation: goodWeather 30s linear infinite;
	}
	@keyframes goodWeather {
		from {
			transform: rotate(0deg);
		}
		to {
			transform: rotate(360deg);
		}
	}
	.loading {
		flex: 1;
		display: grid;
		place-items: center;
		grid-template: 1fr / 1fr;
		font-size: clamp(4rem, 15vw, 30rem);
		letter-spacing: -0.125ch;
		text-align: center;
		background-image: repeating-linear-gradient(
			-30deg,
			var(--loading-stripe) 0vmin,
			var(--loading-stripe) 10vmin,
			transparent 10vmin,
			transparent 20vmin
		);
		background-position: center center;
		background-size: 100% 100%;
		animation: loading 5s linear infinite;
	}
	.location {
		flex: 1;
		display: grid;
		place-items: center;
		grid-template: 1fr / 1fr;
		font-size: clamp(2rem, 5vw, 10rem);
		letter-spacing: -0.125ch;
		text-align: center;
		background-image: repeating-radial-gradient(
			circle,
			transparent 0vmin,
			transparent 15vmin,
			var(--location-stripe) 15vmin,
			var(--location-stripe) 30vmin
		);
		background-position: top center;
		background-size: 100% 100%;
		animation: location 2s ease-in-out infinite alternate;
	}

	@keyframes loading {
		from {
			background-size: 10% 10%;
		}
		to {
			background-size: 160% 160%;
		}
	}
	@keyframes location {
		from {
			background-size: 10% 10%;
		}
		to {
			background-size: 200% 200%;
		}
	}
	@keyframes fade {
		from {
			opacity: 0;
		}
		to {
			opacity: 1;
		}
	}

	/* .forecast {
		animation: fade 1s ease-in-out forwards;
		opacity: 0;
		flex: 0;
		display: grid;
		grid-template-columns: repeat(24, 1fr);
	}
	.hour {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 0.25rem;
		padding-block: 0.5rem;
		font-size: 0.75rem;
	}
	small {
		font-weight: 100;
	}
	.time {
		font-weight: 900;
	}
	.night {
		color: hsla(0deg, 0%, 30%, 0.9);
		background-color: hsla(0deg, 0%, 30%, 0.6);
	} */
</style>
