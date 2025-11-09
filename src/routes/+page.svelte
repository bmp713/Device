

<div class="hidden min-h-screen bg-[#fffe] p-8">
	<div class="max-w-4xl mx-auto">
		<div class="monitor">
			<h1 class="text-6xl text-gray-300 text-center mb-8">Heart Rate Monitor</h1>
		
			<!-- Heart Rate + Glucose Display -->
			<div class="text-center mb-12">
				<div class="text-2xl text-gray-400">
					{getIntensityLabel()}
				</div>
			</div>

			<!-- Exercise Intensity Slider -->
			<div class="mb-8">
				<label for="intensity" class="flex flex-col text-4xl mb-5">
					<h1 class="text-4xl mb-1">Exercise Intensity</h1>
					<h2 class="text-sm text-gray-500">Click different intensity levels to see HR and glucose respond physiologically.</h2>
				</label>
				<div>
					<input
						class="w-full h-3 bg-gray-300 rounded-lg appearance-none cursor-pointer slider"
						type="range"
						id="intensity"
						bind:value={exerciseIntensity}
						oninput={onIntensityChange}
						min="0"
						max="100"
					/>
				</div>
			</div>

			<div class="flex justify-between gap-4 mb-8">
				<button
					onclick={() => isRecording = !isRecording}
					class="px-10 py-3 text-white border border-white-100 rounded-3xl"
				>
					{isRecording ? 'Pause' : 'Record'}
				</button>
				<button
					onclick={clearData}
					class="px-6 py-3 text-white border border-white-600 rounded-3xl"
				>
					Clear Data
				</button>
				
			</div>
			
		</div>		
	</div>
</div>


<style>
	.monitor, .display {
		padding: 50px;
		color: #e0e0e0;
		background: linear-gradient(90deg, #000000, #252525,#000000);
		box-shadow: inset 10px 10px 20px rgba(121, 119, 119, 0.6), 10px 10px 50px #000f;
		border-radius: 100px;
	}	
	.display{
		background: linear-gradient(90deg, #640606, #da1818,#330303);
		box-shadow: inset 10px 10px 20px rgba(39, 39, 39, 0.6), 10px 10px 50px #000f;
	}	
	button{
		cursor: pointer;
	}
	.slider{
		height: 50px;
		padding: 0px;
		border-radius: 20px;
		background: rgb(46, 45, 45);
		box-shadow: inset 10px 10px 20px #000000;
	}
	.slider::-webkit-slider-thumb {
		appearance: none;
		height: 50px;
		width: 50px;
		border-radius: 50%;
		background: #2f2f30;
		box-shadow: inset 10px 10px 20px #9e9a9a, inset 10px 10px 20px #161616, 5px 5px 5px rgb(30, 30, 31);
		border: 5px solid #3a3838;
		cursor: pointer;
	}	
	.slider::-moz-range-thumb {
		height: 50px;
		width: 50px;
		border-radius: 50%;
		background: #020407;
		cursor: pointer;
	}
</style>
