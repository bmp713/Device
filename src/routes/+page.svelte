<script>
	import { onMount } from 'svelte';
	
	// State variables using Svelte 5 runes
	let heartRate = $state(65);
	let exerciseIntensity = $state(50); // 0-100 scale
	
	// numeric font weight for the heart-rate number (toggle between 100 and 300)
	let hrWeight = $state(100);
	let heartBeatData = $state([]);
	let beatInterval = $state(null);
	let hrVariationInterval = $state(null);
	let nextId = $state(1);

	// Recording toggle: when false, heart continues but data is not recorded
	let isRecording = $state(false);

	// timestamp (ms) of last stored record - used to ensure we only record once per second
	let lastRecordAt = $state(0);

	// Simple rate cap: never change more than 3 BPM per second (applied once per second)
	const MAX_HR_CHANGE = 3; // 3 BPM per second
	
	// Exercise intensity ranges
	const intensityRanges = {
		resting: { min: 50, max: 80, label: 'Resting' },
		walking: { min: 80, max: 110, label: 'Walking' },
		jogging: { min: 110, max: 140, label: 'Jogging' },
		sprinting: { min: 140, max: 200, label: 'Sprinting' }
	};
	
	// Get base heart rate from slider position
	function getBaseHeartRate() {
		if (exerciseIntensity <= 25) {
			// Resting (0-25)
			return 50 + (exerciseIntensity / 25) * 30; // 50-80
		} else if (exerciseIntensity <= 50) {
			// Walking (26-50)
			return 80 + ((exerciseIntensity - 25) / 25) * 30; // 80-110
		} else if (exerciseIntensity <= 75) {
			// Jogging (51-75)
			return 110 + ((exerciseIntensity - 50) / 25) * 30; // 110-140
		} else {
			// Sprinting (76-100)
			return 140 + ((exerciseIntensity - 75) / 25) * 60; // 140-200
		}
	}
	
	// Get current intensity label
	function getIntensityLabel() {
		if (exerciseIntensity <= 25) return 'Resting';
		if (exerciseIntensity <= 50) return 'Walking';
		if (exerciseIntensity <= 75) return 'Jogging';
		return 'Sprinting';
	}
	
	// Record heart beat data (throttled to once per second)
	function recordHeartBeat() {
		// Do not record if recording is paused
		if (!isRecording) return;
		const nowTs = Date.now();
		// only record if at least 1 second has passed since last record
		if (nowTs - lastRecordAt < 1000) return;
		lastRecordAt = nowTs;
		const now = new Date(nowTs);
		const data = {
			id: nextId++,
			date: now.toLocaleDateString(),
			time: now.toLocaleTimeString(),
			heartRate: Math.round(heartRate)
		};
		heartBeatData.push(data);

		// Save to localStorage (simulating JSON file storage)
		localStorage.setItem('heartBeatData', JSON.stringify(heartBeatData));
	}
	
	// Load data from localStorage on mount
	function loadSavedData() {
		const saved = localStorage.getItem('heartBeatData');
		if (saved) {
			heartBeatData = JSON.parse(saved);
			if (heartBeatData.length > 0) {
				nextId = Math.max(...heartBeatData.map(d => d.id)) + 1;
			}
		}
	}
	
	// Start heart beat animation
	function startHeartBeat() {
		if (beatInterval) clearInterval(beatInterval);
		
		const bpm = Math.round(heartRate);
		const intervalMs = (60 / bpm) * 1000; // Convert BPM to milliseconds
		
		beatInterval = setInterval(() => {
			// toggle weight between 100 and 300 to create a pulsing effect
			hrWeight = hrWeight === 100 ? 300 : 100;
			if (hrWeight === 300) {
				recordHeartBeat();
			}
		}, intervalMs / 2); // Toggle twice per beat (pulse in/out)
	}
	
	// Start heart rate variation (simple ±5 BPM window around slider base)
	function startHeartRateVariation() {
		if (hrVariationInterval) clearInterval(hrVariationInterval);
		hrVariationInterval = setInterval(() => {
			const baseRate = Math.round(getBaseHeartRate());
			const minRate = Math.max(baseRate - 5, 40);
			const maxRate = Math.min(baseRate + 5, 220);

			// If current heartRate is outside ±5 window, move 1 BPM toward nearest bound.
			if (heartRate < minRate) {
				heartRate = +(Math.min(heartRate + Math.floor(Math.random() * 3) + 1, minRate));
			} else if (heartRate > maxRate) {
				heartRate = +(Math.max(heartRate - Math.floor(Math.random() * 3) + 1, maxRate));
			} else {
				// Inside the window small HR variation
				const step = Math.floor(Math.random() * 3) - 1;
				heartRate = +(Math.max(minRate, Math.min(maxRate, heartRate + step)));
			}
			startHeartBeat();
		}, 1000);
	}
	
	// Handle slider change
	function onIntensityChange() {
		const baseRate = getBaseHeartRate();
		const desired = baseRate + (Math.random() - 0.5) * 10;
		const delta = desired - heartRate;
		const capped = Math.sign(delta) * Math.min(Math.abs(delta), MAX_HR_CHANGE);
		heartRate = +(heartRate + capped).toFixed(2);
		startHeartBeat();
	}
	
	// data is always displayed; no toggle function needed
	
	// Clear all data
	function clearData() {
		heartBeatData = [];
		nextId = 1;
		localStorage.removeItem('heartBeatData');
	}
	
	// Initialize on mount
	onMount(() => {
		loadSavedData();
		onIntensityChange();
		startHeartRateVariation();
		
		// Cleanup on destroy
		return () => {
			if (beatInterval) clearInterval(beatInterval);
			if (hrVariationInterval) clearInterval(hrVariationInterval);
		};
	});
</script>

<div class="min-h-screen bg-[#fffe] p-8">
	<div class="max-w-4xl mx-auto">
		<div class="monitor">
		<h1 class="text-4xl text-gray-300 text-center mb-8">Heart Rate Monitor</h1>
		
		<!-- Heart Rate Display -->
		<div class="text-center mb-12">
			<div class="text-6xl mb-4" style="font-size:50px; color: #f00f;">
				<span style="font-weight: 100;">{Math.round(heartRate)}</span>
				<!-- <span style="font-weight: 300; margin-left:8px; color: #f00f;">bpm</span> -->
			</div>
			<div class="text-2xl text-gray-400">
				{getIntensityLabel()}
			</div>
		</div>
		
		<!-- Exercise Intensity Slider -->
		<div class="mb-8">
			<label for="intensity" class="block text-3xl font-normal text-gray-300 mb-4">
				Exercise Intensity
			</label>
			<div class="relative">
				<input
					type="range"
					id="intensity"
					bind:value={exerciseIntensity}
					oninput={onIntensityChange}
					min="0"
					max="100"
					class="w-full h-3 bg-gray-300 rounded-lg appearance-none cursor-pointer slider"
				/>
				<div class="flex justify-between text-sm text-gray-400 mt-2">
					<span class="text-left">Resting<br>(50-80 BPM)</span>
					<span class="text-center">Walking<br>(80-110 BPM)</span>
					<span class="text-center">Jogging<br>(110-140 BPM)</span>
					<span class="text-right">Sprinting<br>(140-200 BPM)</span>
				</div>
			</div>
		</div>

		<div class="flex gap-4 justify-between mb-8">
			<button
				onclick={() => isRecording = !isRecording}
				class="px-6 py-3 text-white border border-white-600"
			>
				{isRecording ? 'Pause' : 'Start'}
			</button>
			<button
				onclick={clearData}
				class="px-6 py-3 text-white border border-white-600"
			>
				Clear Data
			</button>
			
		</div>
		
		<!-- Data Display (always visible) -->
		<div class="display rounded-lg shadow-lg">
			<h2 class="text-2xl font-normal text-gray-300 mb-4">({heartBeatData.length} records)</h2>
			{#if heartBeatData.length === 0}
				<p class="text-gray-400">No heart beat data recorded yet.</p>
			{:else}
					<div class="overflow-y-auto">
						<pre class="text-sm whitespace-pre-wrap" style="text-align:left;">{JSON.stringify([...heartBeatData].reverse(), null, 2)}</pre>
					</div>
			{/if}
		</div>
	</div>
		<!-- Control Buttons -->
		
	</div>
</div>

<style>
	.monitor, .display {
		padding: 50px;
		color: #e0e0e0;
		background: linear-gradient(90deg, #000000, #1d1d1d,#000000);
		box-shadow: inset 10px 10px 20px rgba(121, 119, 119, 0.6), 10px 10px 50px #000f;
		border-radius: 70px;
	}	
	.display{
		background: linear-gradient(90deg, #640606, #da1818,#330303);
		box-shadow: inset 10px 10px 20px rgba(39, 39, 39, 0.6), 10px 10px 50px #000f;
	}	
	.slider{
		height: 40px;
		padding: 0px;
		border-radius: 20px;
		background: rgb(46, 45, 45);
		box-shadow: inset 10px 10px 20px #000000;
	}
	.slider::-webkit-slider-thumb {
		appearance: none;
		height: 40px;
		width: 40px;
		border-radius: 50%;
		background: #727375;
		box-shadow: inset 10px 10px 20px #9e9a9a;

		cursor: pointer;
	}	
	.slider::-moz-range-thumb {
		height: 40px;
		width: 40px;
		border-radius: 50%;
		background: #020407;
		cursor: pointer;
	}
</style>
