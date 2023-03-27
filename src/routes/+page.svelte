<script>
	/**
	 * @type {boolean}
	 */
	let knownLocation;
	/**
	 * @type {{ hourly: { temperature_2m: any; precipitation: any; windspeed_180m: any; }; daily: { sunrise: any[]; sunset: any[]; }; }}
	 */
	let weatherData;
	/**
	 * @type {any[]}
	 */
	let temp;
	let sunrise;
	let sunset;
	/**
	 * @type {number}
	 */
	let sunriseHour;
	/**
	 * @type {number}
	 */
	let sunsetHour;
	/**
	 * @type {number[]}
	 */
	let precipitation;
	/**
	 * @type {number[]}
	 */
	let windSpeed;

	/**
	 * @param {number} lat
	 * @param {number} lon
	 */
	async function getWeatherData(lat, lon) {
		const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&hourly=temperature_2m,precipitation_probability,precipitation,cloudcover,windspeed_180m&daily=sunrise,sunset&forecast_days=1&timezone=auto`;
		const response = await fetch(url, {});
		if (!response.ok) {
			throw new Error(`Failed to fetch weather data: ${response.status}`);
		}
		setTimeout(async () => {
			weatherData = await response.json();
			temp = weatherData.hourly.temperature_2m;
			sunrise = weatherData.daily.sunrise[0];
			sunset = weatherData.daily.sunset[0];
			sunriseHour = new Date(sunrise).getHours();
			sunsetHour = new Date(sunset).getHours();
			precipitation = weatherData.hourly.precipitation;
			windSpeed = weatherData.hourly.windspeed_180m;
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

	function isGoodWeather() {
		// check if the weather is dry and warm between sunrise and sunset
		for (let i = sunriseHour; i <= sunsetHour; i++) {
			if (temp[i] < 10) {
				return false;
			}
			if (precipitation[i] > 0) {
				return false;
			}
			if (windSpeed[i] > 30) {
				return false;
			}
		}
		return true;
	}

	function showNotification() {
		if ('Notification' in window && Notification.permission === 'granted') {
			navigator.serviceWorker.ready.then((registration) => {
				registration.showNotification('Good weather for hanging washing out!', {
					body: 'The weather tomorrow is going to be dry and warm between 8am and 6pm.',
					icon: 'path/to/icon.png'
				});
			});
		}
	}
</script>

<svelte:head>
	<title>Whirligig Weather</title>
</svelte:head>

<div
	class="wrapper"
	class:good={weatherData && isGoodWeather()}
	class:bad={weatherData && !isGoodWeather()}
>
	<header>
		<h1>Whirligig weather tomorrow?</h1>
	</header>
	{#if knownLocation}
		{#if weatherData}
			<div class="message">
				{#if isGoodWeather()}
					<p>Yup!</p>
				{:else}
					<p>Nope.</p>
				{/if}
			</div>
			<!-- <div class="forecast">
                {#each precipitation as rain, i}
                    {#if i <= sunriseHour || i >= sunsetHour}
                        <div class="hour night">
                            <span class="time">{i + 1}</span>
                            <span class="rain">{rain}<small>mm</small></span>
                            <span class="temp">{temp[i]}<small>℃</small></span>
                        </div>
                    {:else}
                        <div class="hour">
                            <span class="time">{i + 1}</span>
                            <span class="rain">{rain}<small>mm</small></span>
                            <span class="temp">{temp[i]}<small>℃</small></span>
                        </div>
                    {/if}
                {/each}
            </div> -->
		{:else}
			<div class="loading"><p>Checking...</p></div>
		{/if}
	{:else}
		<div class="location">
			<p>Where are you?</p>
		</div>
	{/if}
</div>

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
		--loading-stripe: hsl(180, 100%, 17.5%);
		--location-stripe: hsl(190, 75%, 50%);

		--text: white;
		--text-shadow: 0 0.25em 0 #000;
		font-family: 'Space Grotesk', sans-serif;
		line-height: 0.9;
		color: var(--text);
		font-weight: 700;
		text-shadow: var(--text-shadow);
	}
	.wrapper {
		display: flex;
		flex-direction: column;
		height: 100vh;
		background-color: var(--default-background);
		transition: all 450ms linear;
	}
	header h1 {
		font-size: 3rem;
		padding: 2rem;
		margin: 0;
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
			0 0 0 56vmax var(--good-line), 0 0 0 60vmax var(--good-background);
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
