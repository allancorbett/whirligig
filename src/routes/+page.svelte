<script lang="ts">
	let knownLocation: boolean;
	let demoDialog: HTMLDialogElement;
	let weatherData: {
		hourly: { apparent_temperature: any; precipitation: any; windspeed_180m: any };
		daily: { sunrise: any[]; sunset: any[] };
	};
	let temp: number[];
	let day1Sunrise: number;
	let day1Sunset: number;
	let day1Noon: number;
	let day2Sunrise: number;
	let day2Sunset: number;
	let day2Noon: number;
	let day3Sunrise: number;
	let day3Sunset: number;
	let day3Noon: number;
	let headerColour: string | undefined;
	let precipitation: number[];
	let windSpeed: number[];
	const noon = 12;
	const day = 24;

	async function getWeatherData(lat: number, lon: number) {
		const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&hourly=apparent_temperature,precipitation_probability,precipitation,cloudcover,windspeed_180m&daily=sunrise,sunset&forecast_days=3&timezone=auto`;
		const response = await fetch(url, {});
		if (!response.ok) {
			throw new Error(`Failed to fetch weather data: ${response.status}`);
		}
		setTimeout(async () => {
			weatherData = await response.json();
			temp = weatherData.hourly.apparent_temperature;
			day1Sunrise = new Date(weatherData.daily.sunrise[0]).getHours();
			day1Noon = noon;
			day1Sunset = new Date(weatherData.daily.sunset[0]).getHours();
			day2Sunrise = new Date(weatherData.daily.sunrise[1]).getHours() + day;
			day2Noon = noon + day;
			day2Sunset = new Date(weatherData.daily.sunset[1]).getHours() + day;
			day3Sunrise = new Date(weatherData.daily.sunrise[2]).getHours() + day * 2;
			day3Noon = noon + day * 2;
			day3Sunset = new Date(weatherData.daily.sunset[2]).getHours() + day * 2;
			precipitation = weatherData.hourly.precipitation;
			windSpeed = weatherData.hourly.windspeed_180m;
			headerColour = determineHeaderColour();
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

	function day1Am() {
		for (let i = day1Sunrise + 2; i <= day1Noon; i++) {
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
	function day1Pm() {
		for (let i = day1Noon; i <= day1Sunset; i++) {
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
	function day2Am() {
		for (let i = day2Sunrise + 2; i <= day2Noon; i++) {
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
	function day2Pm() {
		for (let i = day2Noon; i <= day2Sunset; i++) {
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
	function day3Am() {
		for (let i = day3Sunrise + 2; i <= day3Noon; i++) {
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
	function day3Pm() {
		for (let i = day3Noon; i <= day3Sunset; i++) {
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

	function isDay1Am() {
		const now = new Date().getHours();
		return now < day1Noon;
	}

	function isDay1Pm() {
		const now = new Date().getHours();
		return now < day1Sunset;
	}

	function determineHeaderColour() {
		if (isDay1Am()) {
			return day1Am() ? 'good day1am' : 'bad day1am';
		} else if (!isDay1Am() && isDay1Pm()) {
			return day1Pm() ? 'good day1pm' : 'bad day1pm';
		} else if (!isDay1Am() && !isDay1Pm()) {
			return day2Am() ? 'good day2am' : 'bad day2am';
		}
	}
</script>

<svelte:head>
	<title>Whirligig Weather</title>
	<meta property="og:title" content="Whirligig - Should you hang your washing out to dry?" />
	<meta
		property="og:description"
		content="Whirligig is a website that helps you determine whether or not you should hang your washing out to dry based on the weather in your area."
	/>
	<meta
		property="og:image"
		content="https://allancorbett.github.io/whirligig/whirligig-image.jpg"
	/>
	<meta property="og:url" content="https://allancorbett.github.io/whirligig/whirligig" />
	<meta property="og:type" content="website" />

	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
	<link
		href="https://fonts.googleapis.com/css2?family=Syne:wght@700;800&display=swap"
		rel="stylesheet"
	/>
</svelte:head>
<main class={headerColour}>
	<header>
		<h1>Whirligig weather?</h1>
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

				<button on:click={() => demoDialog.close()}>Close & Save</button>
			</div>
		</dialog>
	</header>
	{#key minTemp + minRain + maxWind}
		<div class="wrapper">
			{#if knownLocation}
				{#if weatherData}
					{#if isDay1Am()}
						<div class="message {day1Am() ? 'good' : 'bad'}">
							<div class="time">this morning?</div>
							<div class="status">{day1Am() ? 'Yup!' : 'Nope'}</div>
						</div>
					{/if}
					{#if isDay1Pm()}
						<div class="message {day1Pm() ? 'good' : 'bad'}">
							<div class="time">this afternoon?</div>
							<div class="status">{day1Pm() ? 'Yup!' : 'Nope'}</div>
						</div>
					{/if}
					<div class="message {day2Am() ? 'good' : 'bad'}">
						<div class="time">tomorrow morning?</div>
						<div class="status">{day2Am() ? 'Yup!' : 'Nope'}</div>
					</div>
					<div class="message {day2Pm() ? 'good' : 'bad'}">
						<div class="time">tomorrow afternoon?</div>
						<div class="status">{day2Pm() ? 'Yup!' : 'Nope'}</div>
					</div>
					<div class="message {day3Am() ? 'good' : 'bad'}">
						<div class="time">the day after morning?</div>
						<div class="status">{day3Am() ? 'Yup!' : 'Nope'}</div>
					</div>
					<div class="message {day3Pm() ? 'good' : 'bad'}">
						<div class="time">the day after afternoon?</div>
						<div class="status">{day3Pm() ? 'Yup!' : 'Nope'}</div>
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

<style>
	:root {
		--shadow-color: 0, 0%, 0%;
		--shadow: 0px 1px 1.1px hsla(var(--shadow-color), 0.13),
			0px 6.2px 6.7px -0.5px hsla(var(--shadow-color), 0.12),
			0px 12.2px 13.3px -0.9px hsla(var(--shadow-color), 0.12),
			0px 21.9px 23.8px -1.4px hsla(var(--shadow-color), 0.12),
			0px 38.1px 41.4px -1.8px hsla(var(--shadow-color), 0.11),
			0px 63.6px 69.2px -2.3px hsla(var(--shadow-color), 0.11),
			0px 101px 109.8px -2.7px hsla(var(--shadow-color), 0.11);
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
		--text-shadow: 0 0.25em 0 hsl(0, 0%, 0%);
		font-family: 'Syne', sans-serif;
		line-height: 1;
		color: var(--text);
		font-weight: 700;
		text-shadow: var(--text-shadow);
	}

	main {
		display: flex;
		flex-direction: column;
		background-color: var(--default-background);
		min-height: 100vh;
	}
	header {
		z-index: 2;
		backdrop-filter: blur(2rem);
		--webkit-backdrop-filter: blur(2rem);
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 2rem;
		box-shadow: var(--shadow);
		font-weight: 800;
	}
	header h1 {
		font-size: clamp(2rem, 2vw, 5rem);
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
		background-color: var(--default-background);
	}
	@supports (backdrop-filter: blur(2rem)) {
		dialog::backdrop {
			backdrop-filter: blur(2rem);
			--webkit-backdrop-filter: blur(2rem);
		}
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
		font-weight: 700;
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
	}
	dialog button:hover {
		background-color: black;
		color: white;
		box-shadow: none;
		transform: translateY(0.25rem);
	}

	.wrapper {
		display: flex;
		flex-direction: column;
		transition: all 450ms linear;
		flex: 1;
		gap: 6vmax;
		margin: 0;
	}
	.message {
		box-shadow: var(--shadow);
		flex: 1 0 auto;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		aspect-ratio: 5/2;
		border-radius: 1vmin 5vmin 10vmin;
		margin-inline: 2vmax;
		overflow: hidden;
		z-index: 1;
	}
	.message:first-of-type {
		margin-top: 6vmax;
	}

	@media (max-width: 960px) {
		.message {
			aspect-ratio: 5/4;
		}
	}
	.message .status {
		margin: 0;
		font-size: clamp(2rem, 20vw, 15rem);
		font-weight: 800;
		letter-spacing: -0.05ch;
	}
	.message .time {
		font-size: clamp(1rem, 5vw, 5rem);
	}
	.bad {
		background-color: var(--bad-background);
	}
	.message.bad {
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

	.message.good::before {
		content: '';
		place-self: center;
		z-index: -1;
		height: 0;
		width: 0;
		box-shadow: 0 0 0 3vmax var(--good-line), 0 0 0 10vmax var(--good-background),
			0 0 0 13vmax var(--good-line), 0 0 0 20vmax var(--good-background),
			0 0 0 23vmax var(--good-line), 0 0 0 30vmax var(--good-background),
			0 0 0 33vmax var(--good-line), 0 0 0 40vmax var(--good-background),
			0 0 0 43vmax var(--good-line), 0 0 0 50vmax var(--good-background),
			0 0 0 53vmax var(--good-line), 0 0 0 60vmax var(--good-background),
			0 0 0 63vmax var(--good-line), 0 0 0 70vmax var(--good-background),
			0 0 0 73vmax var(--good-line), 0 0 0 80vmax var(--good-background),
			0 0 0 83vmax var(--good-line), 0 0 0 90vmax var(--good-background),
			0 0 0 93vmax var(--good-line), 0 0 0 100vmax var(--good-background);
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
	.loading,
	.location {
		flex: 1;
		display: grid;
		place-items: center;
		grid-template: 1fr / 1fr;
		text-align: center;
		background-size: 100% 100%;
		font-weight: 700;
		font-size: clamp(2rem, 13vw, 10rem);
	}
	.loading {
		background-image: repeating-linear-gradient(
			-30deg,
			var(--loading-stripe) 0vmin,
			var(--loading-stripe) 10vmin,
			transparent 10vmin,
			transparent 20vmin
		);
		background-position: center center;
		animation: loading 5s linear infinite;
	}
	.location {
		background-image: repeating-radial-gradient(
			circle,
			transparent 0vmin,
			transparent 15vmin,
			var(--location-stripe) 15vmin,
			var(--location-stripe) 30vmin
		);
		background-position: top center;
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
</style>
