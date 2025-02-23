<script lang="ts">
	import LicenseSelector from '../LicenseSelector.svelte';
	import ResultPreview from '../ResultPreview.svelte';
	import { urlHashStore } from '$lib/url-hash-store';
	import { countVariables } from '$lib/meteo';
	import AccordionItem from '$lib/Elements/AccordionItem.svelte';
	import { fade } from 'svelte/transition';
	import { CalendarEvent, Clock } from 'svelte-bootstrap-icons';
	import StartEndDate from '../StartEndDate.svelte';
	import LocationSelection from '../LocationSelection.svelte';

	const defaultParameter = {
		daily: [],
		location_mode: 'location_search',
		csv_coordinates: '',
		timeformat: 'iso8601',
		past_days: '0',
		forecast_days: '92',
		start_date: '',
		end_date: '',
		time_mode: 'forecast_days',
		models: [],
		ensemble: false
	};

	const params = urlHashStore({
		latitude: [59.91],
		longitude: [10.75],
		...defaultParameter,
		daily: ['river_discharge']
	});

	const daily = [
		[
			{ name: 'river_discharge', label: 'Débit fluvial' },
			{ name: 'river_discharge_mean', label: 'Débit moyen de la rivière' },
			{ name: 'river_discharge_median', label: 'Médiane du débit fluvial' },
			{ name: 'river_discharge_max', label: 'Débit maximal de la rivière' },
			{ name: 'river_discharge_min', label: 'Débit minimum de la rivière' },
			{ name: 'river_discharge_p25', label: 'Débit fluvial, 25<sup>ème</sup> centile' },
			{ name: 'river_discharge_p75', label: 'Débit fluvial, 75<sup>ème</sup> centile' }
		]
	];

	const models = [
		[
			{ name: 'seamless_v4', label: 'GloFAS v4 Seamless' },
			{ name: 'forecast_v4', label: 'GloFAS v4 Forecast' },
			{ name: 'consolidated_v4', label: 'GloFAS v4 Consolidated' }
		],
		[
			{ name: 'seamless_v3', label: 'GloFAS v3 Seamless' },
			{ name: 'forecast_v3', label: 'GloFAS v3 Forecast' },
			{ name: 'consolidated_v3', label: 'GloFAS v3 Consolidated' }
		]
	];
</script>

<svelte:head>
	<title>MNOCC - FLOOD | MNOCC</title>
	<link rel="canonical" href="https://open-meteo.com/en/docs/flood-api" />
	<meta
		name="description"
		content="Access ensemble flood forecasts estimating the volume of water discharged by rivers worldwide. Get invaluable information for predicting flood events and taking proactive measures for flood preparedness. Ensure safety and minimize the impact of floods with our reliable flood data."
	/>
</svelte:head>

<form method="get" action="https://flood-api.open-meteo.com/v1/flood">
	<LocationSelection
		bind:latitude={$params.latitude}
		bind:longitude={$params.longitude}
		bind:location_mode={$params.location_mode}
		bind:csv_coordinates={$params.csv_coordinates}
	/>

	<div class="row py-3 px-0">
		<div>
			<ul class="nav nav-underline" id="pills-tab" role="tablist">
				<li class="nav-item" role="presentation" style="width: 70px;">
					<span class="nav-link disabled" aria-disabled="true">Time:</span>
				</li>
				<li class="nav-item" role="presentation">
					<button
						class="nav-link"
						class:active={$params.time_mode == 'forecast_days'}
						id="pills-forecast_days-tab"
						type="button"
						role="tab"
						aria-controls="pills-forecast_days"
						aria-selected="true"
						on:click={() => ($params.time_mode = 'forecast_days')}><Clock/> Forecast Length</button
					>
				</li>
				<li class="nav-item" role="presentation">
					<button
						class="nav-link"
						class:active={$params.time_mode == 'time_interval'}
						id="pills-time_interval-tab"
						type="button"
						role="tab"
						aria-controls="pills-time_interval"
						on:click={() => ($params.time_mode = 'time_interval')}
						aria-selected="true"><CalendarEvent/> Time Interval</button
					>
				</li>
			</ul>
		</div>
		<div class="tab-content py-3" id="pills-tabContent">
			{#if $params.time_mode == 'forecast_days'}
				<div
					class="tab-pane active"
					in:fade
					id="pills-forecast_days"
					role="tabpanel"
					aria-labelledby="pills-forecast_days-tab"
					tabindex="0"
				>
					<div class="row">
						<div class="col-md-3">
							<div class="form-floating mb-3">
								<select
									class="form-select"
									name="forecast_days"
									id="forecast_days"
									aria-label="Forecast days"
									bind:value={$params.forecast_days}
								>
									<option value="1">1 day</option>
									<option value="7">7 days</option>
									<option value="14">2 weeks</option>
									<option value="31">1 month</option>
									<option value="92">3 months (default)</option>
									<option value="183">6 months</option>
								</select>
								<label for="forecast_days">Forecast days</label>
							</div>
						</div>
						<div class="col-md-3">
							<div class="form-floating mb-3">
								<select
									class="form-select"
									name="past_days"
									id="past_days"
									aria-label="Past days"
									bind:value={$params.past_days}
								>
									<option value="0">0 (default)</option>
									<option value="1">1</option>
									<option value="2">2</option>
									<option value="3">3</option>
									<option value="5">5</option>
									<option value="7">1 week</option>
									<option value="14">2 weeks</option>
									<option value="31">1 month</option>
									<option value="61">2 months</option>
									<option value="92">3 months</option>
								</select>
								<label for="past_days">Past days</label>
							</div>
						</div>
					</div>
				</div>
			{/if}
			{#if $params.time_mode == 'time_interval'}
				<div
					class="tab-pane active"
					in:fade
					id="pills-time_interval"
					role="tabpanel"
					aria-labelledby="pills-time_interval-tab"
					tabindex="0"
				>
					<div class="row">
						<div class="col-md-6 mb-3">
							<StartEndDate bind:start_date={$params.start_date} bind:end_date={$params.end_date}/>
						</div>
					</div>
				</div>
			{/if}
		</div>
	</div>

	<div class="row py-3 px-0">
		<h2>Variables météorologiques</h2>
		{#each daily as group}
			<div class="col-md-6">
				{#each group as e}
					<div class="form-check">
						<input
							class="form-check-input"
							type="checkbox"
							value={e.name}
							id="{e.name}_daily"
							name="daily"
							bind:group={$params.daily}
						/>
						<label class="form-check-label" for="{e.name}_daily">{@html e.label}</label>
					</div>
				{/each}
			</div>
		{/each}
		<div class="col-md-6">
			<div class="form-check form-switch">
				<input
					class="form-check-input"
					type="checkbox"
					id="ensemble"
					name="ensemble"
					value="true"
					bind:checked={$params.ensemble}
				/>
				<label class="form-check-label" for="ensemble">All 50 Ensemble Members</label>
			</div>
			<p class="mt-2">
				<small class="text-muted"
					>Note: Statistical and ensemble forecasts are only available for forecasts.</small
				>
			</p>
		</div>
	</div>

	<div class="row py-3 px-0">
		<div class="accordion" id="accordionVariables">
			<AccordionItem
				id="models"
				title="Flood Models"
				count={countVariables(models, $params.models)}
			>
				{#each models as group}
					<div class="col-md-4 mb-3">
						{#each group as e}
							<div class="form-check">
								<input
									class="form-check-input"
									type="checkbox"
									value={e.name}
									id="{e.name}_model"
									name="models"
									bind:group={$params.models}
								/>
								<label class="form-check-label" for="{e.name}_model">{e.label}</label>
							</div>
						{/each}
					</div>
				{/each}
				<div class="col-md-12">
					<p>
						<!-- <small class="text-muted"
							>Note: <mark>Seamless</mark> combines forecast and consolidated historical data. Per default,
							GloFAS version 4 with data from 1984 up to 7 months of forecast is used.</small
						> -->
					</p>
				</div>
			</AccordionItem>
		</div>
	</div>

	<div class="row py-3 px-0">
		<h2>Settings</h2>
		<div class="col-md-3">
			<div class="form-floating mb-3">
				<select
					class="form-select"
					name="timeformat"
					id="timeformat"
					aria-label="Timeformat"
					bind:value={$params.timeformat}
				>
					<option value="iso8601">ISO 8601 (e.g. 2022-12-31)</option>
					<option value="unixtime">Unix timestamp</option>
				</select>
				<label for="timeformat">Timeformat</label>
			</div>
		</div>
	</div>

	<LicenseSelector />
</form>

<ResultPreview {params} {defaultParameter} type="flood" action="flood" />

<div class="col-12 py-5">
	<h2 id="data-sources">Sources de données</h2>
	<!-- <p>
		This API uses reanalysis and forecast data from the <a
			href="https://www.globalfloods.eu"
			target="_blank">Global Flood Awareness System (GloFAS)</a
		>. Per default, GloFAS version 4 with seamless data from 1984 until 7 months of forecast is
		used.
	</p> -->
	<!-- <p>
		Please note: Due to the 5 km resolution the closest river might not be selected correctly.
		Varying coordiantes by 0.1° can help to get a more representable discharge rate. The GloFAS
		website provides additional maps to help understand how rivers are covered in this dataset.
	</p> -->
	<div class="table-responsive">
		<table class="table">
			<thead>
				<tr>
					<th scope="col">Weather Model</th>
					<th scope="col">Region</th>
					<th scope="col">Spatial Resolution</th>
					<th scope="col">Temporal Resolution</th>
					<th scope="col">Data Length</th>
					<th scope="col">Update frequency</th>
				</tr>
			</thead>
			<tbody>
				<tr>
					<th scope="row"
						><a
							href="https://cds.climate.copernicus.eu/cdsapp#!/dataset/cems-glofas-historical?tab=overview"
							target="_blank">GloFAS v4 Reanalysis</a
						></th
					>
					<td>Global</td>
					<td>0.05° (~5 km)</td>
					<td>Daily</td>
					<td>1984 - July 2022</td>
					<td>-</td>
				</tr>
				<tr>
					<th scope="row"><a
						href="https://cds.climate.copernicus.eu/cdsapp#!/dataset/cems-glofas-forecast?tab=overview"
						target="_blank">GloFAS v4 Forecast</a
					></th>
					<td>Global</td>
					<td>0.05° (~5 km)</td>
					<td>Daily</td>
					<td>30 days forecast</td>
					<td>Daily</td>
				</tr>
				<tr>
					<th scope="row"><a
						href="https://cds.climate.copernicus.eu/cdsapp#!/dataset/cems-glofas-seasonal?tab=overview"
						target="_blank">GloFAS v4 Seasonal Forecast</a
					></th>
					<td>Global</td>
					<td>0.05° (~5 km)</td>
					<td>Daily</td>
					<td>7 months forecast</td>
					<td>Monthly</td>
				</tr>
				<tr>
					<th scope="row"
						><a
							href="https://cds.climate.copernicus.eu/cdsapp#!/dataset/cems-glofas-historical?tab=overview"
							target="_blank">GloFAS v3 Reanalysis</a
						></th
					>
					<td>Global</td>
					<td>0.1° (~11 km)</td>
					<td>Daily</td>
					<td>1984 - July 2022</td>
					<td>-</td>
				</tr>
				<tr>
					<th scope="row"
						><a
							href="https://cds.climate.copernicus.eu/cdsapp#!/dataset/cems-glofas-forecast?tab=overview"
							target="_blank">GloFAS v3 Forecast</a
						></th
					>
					<td>Global</td>
					<td>0.1° (~11 km)</td>
					<td>Daily</td>
					<td>30 days forecast</td>
					<td>Daily</td>
				</tr>
				<tr>
					<th scope="row"
						><a
							href="https://cds.climate.copernicus.eu/cdsapp#!/dataset/cems-glofas-seasonal?tab=overview"
							target="_blank">GloFAS v3 Seasonal Forecast</a
						></th
					>
					<td>Global</td>
					<td>0.1° (~11 km)</td>
					<td>Daily</td>
					<td>7 months forecast</td>
					<td>Monthly</td>
				</tr>
			</tbody>
		</table>
	</div>

	<!-- <h2 id="api-documentation" class="mt-5">API Documentation</h2>
	<p>
		The API endpoint <mark>/v1/flood</mark> accepts a geographical coordinate and returns river discharge
		data from the largest river in a 5 km area for the given coordinates. All URL parameters are listed
		below:
	</p> -->
	<!-- <div class="table-responsive">
		<table class="table">
			<thead>
				<tr>
					<th scope="col">Parameter</th>
					<th scope="col">Format</th>
					<th scope="col">Required</th>
					<th scope="col">Default</th>
					<th scope="col">Description</th>
				</tr>
			</thead>
			<tbody>
				<tr>
					<th scope="row">latitude, longitude</th>
					<td>Floating point</td>
					<td>Yes</td>
					<td />
					<td
						>Geographical WGS84 coordinates of the location. Multiple coordinates can be comma
						separated. E.g. <mark>&latitude=52.52,48.85&longitude=13.41,2.35</mark>. To return data
						for multiple locations the JSON output changes to a list of structures. CSV and XLSX
						formats add a column <mark>location_id</mark>.</td
					>
				</tr>
				<tr>
					<th scope="row">daily</th>
					<td>String array</td>
					<td>No</td>
					<td />
					<td
						>A list of weather variables which should be returned. Values can be comma separated, or
						multiple
						<mark>&daily=</mark> parameter in the URL can be used.
					</td>
				</tr>
				<tr>
					<th scope="row">timeformat</th>
					<td>String</td>
					<td>No</td>
					<td><mark>iso8601</mark></td>
					<td
						>If format <mark>unixtime</mark> is selected, all time values are returned in UNIX epoch
						time in seconds. Please note that all time is then in GMT+0!
					</td>
				</tr>
				<tr>
					<th scope="row">past_days</th>
					<td>Integer</td>
					<td>No</td>
					<td><mark>0</mark></td>
					<td>If <mark>past_days</mark> is set, past data can be returned.</td>
				</tr>
				<tr>
					<th scope="row">forecast_days</th>
					<td>Integer (0-210)</td>
					<td>No</td>
					<td><mark>92</mark></td>
					<td>Per default, only 92 days are returned. Up to 210 days of forecast are possible.</td>
				</tr>
				<tr>
					<th scope="row">start_date<br />end_date</th>
					<td>String (yyyy-mm-dd)</td>
					<td>No</td>
					<td />
					<td
						>The time interval to get data. A day must be specified as an ISO8601 date (e.g.
						<mark>2022-06-30</mark>). Data are available from 1984-01-01 until 7 month forecast.
					</td>
				</tr>
				<tr>
					<th scope="row">ensemble</th>
					<td>Boolean</td>
					<td>No</td>
					<td />
					<td>If <mark>True</mark> all forecast ensemble members will be returned</td>
				</tr>
				<tr>
					<th scope="row">cell_selection</th>
					<td>String</td>
					<td>No</td>
					<td><mark>nearest</mark></td>
					<td
						>Set a preference how grid-cells are selected. The default <mark>land</mark> finds a
						suitable grid-cell on land with
						<a
							href="https://openmeteo.substack.com/p/improving-weather-forecasts-with"
							title="Elevation based grid-cell selection explained"
							>similar elevation to the requested coordinates using a 90-meter digital elevation
							model</a
						>.
						<mark>sea</mark> prefers grid-cells on sea. <mark>nearest</mark> selects the nearest possible
						grid-cell.
					</td>
				</tr>
				<tr>
					<th scope="row">apikey</th>
					<td>String</td>
					<td>No</td>
					<td />
					<td
						>Only required to commercial use to access reserved API resources for customers. The
						server URL requires the prefix <mark>customer-</mark>. See
						<a href="/en/pricing" title="Pricing information to use the weather API commercially"
							>pricing</a
						> for more information.</td
					>
				</tr>
			</tbody>
		</table>
	</div> -->
	<!-- <p>
		Additional optional URL parameters will be added. For API stability, no required parameters will
		be added in the future!
	</p> -->

	<!-- <h3 class="mt-5">Daily Parameter Definition</h3>
	<p>The parameter <mark>&daily=</mark> accepts the following values:</p>
	<div class="table-responsive">
		<table class="table">
			<thead>
				<tr>
					<th scope="col">Variable</th>
					<th scope="col">Unit</th>
					<th scope="col">Description</th>
				</tr>
			</thead>
			<tbody>
				<tr>
					<th scope="row">river_discharge</th>
					<td>m³/s</td>
					<td>Daily river discharge rate in m³/s</td>
				</tr>
				<tr>
					<th scope="row"
						>river_discharge_mean<br />river_discharge_median<br />river_discharge_max<br
						/>river_discharge_min<br />river_discharge_p25<br />river_discharge_p75</th
					>
					<td>m³/s</td>
					<td
						>Statistical analysis from ensemble members for river discharge rate in m³/s. Only
						available for forecasts and not for consolidated historical data.</td
					>
				</tr>
			</tbody>
		</table>
	</div>

	<h3 class="mt-5">JSON Return Object</h3>
	<p>On success a JSON object will be returned.</p>
	<pre>
      <code>
{`
  "latitude": 52.52,
  "longitude": 13.419,
  "generationtime_ms": 2.2119,
  "timezone": "Europe/Berlin",
  "timezone_abbreviation": "CEST",
  "hourly": {
    "time": ["2022-07-01T00:00", "2022-07-01T01:00", "2022-07-01T02:00", ...],
    "temperature_2m": [13, 12.7, 12.7, 12.5, 12.5, 12.8, 13, 12.9, 13.3, ...]
  },
  "hourly_units": {
    "temperature_2m": "°C"
  },
`}
      </code>
    </pre>
	<div class="table-responsive">
		<table class="table">
			<thead>
				<tr>
					<th scope="col">Parameter</th>
					<th scope="col">Format</th>
					<th scope="col">Description</th>
				</tr>
			</thead>
			<tbody>
				<tr>
					<th scope="row">latitude, longitude</th>
					<td>Floating point</td>
					<td
						>WGS84 of the center of the weather grid-cell which was used to generate this forecast.
						This coordinate might be a few kilometers away from the requested coordinate.</td
					>
				</tr>
				<tr>
					<th scope="row">generationtime_ms</th>
					<td>Floating point</td>
					<td
						>Generation time of the weather forecast in milliseconds. This is mainly used for
						performance monitoring and improvements.</td
					>
				</tr>
				<tr>
					<th scope="row">daily</th>
					<td>Object</td>
					<td
						>For each selected weather variable, data will be returned as a floating point array.
						Additionally a
						<mark>time</mark> array will be returned with ISO8601 timestamps.
					</td>
				</tr>
				<tr>
					<th scope="row">daily_units</th>
					<td>Object</td>
					<td>For each selected weather variable, the unit will be listed here.</td>
				</tr>
			</tbody>
		</table>
	</div>
	<h3 class="mt-5">Errors</h3>
	<p>
		In case an error occurs, for example a URL parameter is not correctly specified, a JSON error
		object is returned with a HTTP 400 status code.
	</p>
	<pre>
      <code>
{`
  "error": true,
  "reason": "Cannot initialize WeatherVariable from invalid String value tempeture_2m for key hourly"
`}
      </code>
    </pre> -->
</div>
