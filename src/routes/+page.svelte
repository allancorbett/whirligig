<script>
	// @ts-nocheck
	let weatherData;
	let temp;
	let sunrise;
	let sunset;
	let sunriseHour;
	let sunsetHour;
	let precipitation;
	let windSpeed;

	async function getWeatherData(lat, lon) {
		const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&hourly=temperature_2m,precipitation_probability,precipitation,cloudcover,windspeed_180m&daily=sunrise,sunset&forecast_days=1&timezone=auto`;
		const response = await fetch(url, {});
		if (!response.ok) {
			throw new Error(`Failed to fetch weather data: ${response.status}`);
		}
		// const data = await response.json();
		weatherData = await response.json();
		temp = weatherData.hourly.temperature_2m;
		sunrise = weatherData.daily.sunrise[0];
		sunset = weatherData.daily.sunset[0];
		sunriseHour = new Date(sunrise).getHours();
		sunsetHour = new Date(sunset).getHours();
		precipitation = weatherData.hourly.precipitation;
		windSpeed = weatherData.hourly.windspeed_180m;
	}

	if (typeof navigator !== 'undefined') {
		navigator.geolocation.getCurrentPosition(
			async (position) => {
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

	function isGoodWeather(weatherData) {
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

<div class="wrapper">
	{#if weatherData}
		<div class="message">
			{#if isGoodWeather(weatherData)}
				<p class="good">The weather is good for hanging washing out tomorrow!</p>
			{:else}
				<p class="bad">The weather is not good for hanging washing out tomorrow.</p>
			{/if}
		</div>
		<div class="forecast">
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
		</div>
	{:else}
		<div class="loading">Loading weather data...</div>
	{/if}
</div>
<link rel="preconnect" href="https://rsms.me/" />
<link rel="stylesheet" href="https://rsms.me/inter/inter.css" />

<style>
	:root {
		font-family: 'Inter', sans-serif;
		background-color: hsl(194.7, 53.3%, 79%);
	}
	@supports (font-variation-settings: normal) {
		:root {
			font-family: 'Inter var', sans-serif;
		}
	}
	.wrapper {
		display: flex;
		flex-direction: column;
		height: 100vh;
	}

	.message {
		font-weight: 900;
		flex: 1;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 3rem;
		opacity: 0;
		animation: fade 1s ease-in-out forwards;
	}
	.good {
		color: green;
	}
	.bad {
		color: red;
	}
	.loading {
		flex: 1;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 1rem;
	}
	@keyframes fade {
		from {
			opacity: 0;
		}
		to {
			opacity: 1;
		}
	}

	.forecast {
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
	}
</style>
