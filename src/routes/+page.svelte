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

	// timestamp (ms) of last stored record - used to ensure we only record once per second
	let lastRecordAt = $state(0);
	
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
	
	// Start heart rate variation
	function startHeartRateVariation() {
		if (hrVariationInterval) clearInterval(hrVariationInterval);
		
		hrVariationInterval = setInterval(() => {
			const baseRate = getBaseHeartRate();
			const variation = (Math.random() - 0.5) * 20; // ±10 BPM variation
			const direction = Math.random() > 0.5 ? 1 : -1;
			
			// Change by 1 BPM up or down
			heartRate += direction;
			
			// Keep within reasonable bounds around base rate
			const minRate = Math.max(baseRate - 10, 40);
			const maxRate = Math.min(baseRate + 10, 220);
			
			if (heartRate < minRate) heartRate = minRate;
			if (heartRate > maxRate) heartRate = maxRate;
			
			// Restart beat interval with new rate
			startHeartBeat();
		}, 1000); // Change every second
	}
	
	// Handle slider change
	function onIntensityChange() {
		const baseRate = getBaseHeartRate();
		heartRate = baseRate + (Math.random() - 0.5) * 10; // Start with some variation
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

<div class="min-h-screen bg-[#000e] p-8">
	<div class="max-w-4xl mx-auto">
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
			<label for="intensity" class="block text-xl font-semibold text-gray-300 mb-4">
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
		
		<!-- Control Buttons -->
		<div class="flex gap-4 justify-end mb-8">
			<button
				onclick={clearData}
				class="px-6 py-3 text-white border border-white-600 hover:bg-red-700"
			>
				Clear Data
			</button>
		</div>
		
		<!-- Data Display (always visible) -->
		<div class="display rounded-lg shadow-lg p-6">
			<h2 class="text-2xl font-bold text-gray-300 mb-4">Heart Beat Data ({heartBeatData.length} records)</h2>
			{#if heartBeatData.length === 0}
				<p class="text-gray-400">No heart beat data recorded yet.</p>
			{:else}
					<div class="overflow-y-auto">
						<pre class="text-sm p-4 whitespace-pre-wrap">
							{JSON.stringify([...heartBeatData].reverse(), null, 2)}
						</pre>
					</div>
			{/if}
		</div>
	</div>
</div>

<style>
	.display {
		background: linear-gradient(90deg, #330303, #8f1010,#330303);
		color: #e0e0e0;
		font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;	
	}	
	.slider::-webkit-slider-thumb {
		appearance: none;
		height: 25px;
		width: 25px;
		border-radius: 50%;
		background: #181c24;
		cursor: pointer;
		border: 2px solid #ffffff;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
	}
	
	.slider::-moz-range-thumb {
		height: 25px;
		width: 25px;
		border-radius: 50%;
		background: #020407;
		cursor: pointer;
		border: 2px solid #ffffff;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
	}
</style>
