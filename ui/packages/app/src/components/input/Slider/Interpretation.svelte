<script>
	import { getSaliencyColor } from "../../utils/helpers";

	export let value, interpretation, theme;
	export let minimum, maximum, step;
</script>

<div class="input-slider text-center" {theme}>
	<input
		type="range"
		class="range w-full appearance-none transition rounded h-4"
		disabled
		{value}
		min={minimum}
		max={maximum}
		{step}
	/>
	<div class="interpret_range flex">
		{#each interpretation as interpret_value}
			<div
				class="flex-1 h-4"
				style={"background-color: " + getSaliencyColor(interpret_value)}
			/>
		{/each}
	</div>
	<div class="value inline-block mx-auto mt-1 px-2 py-0.5 rounded">{value}</div>
</div>

<style lang="postcss">
	.range::-webkit-slider-thumb {
		-webkit-appearance: none;
		@apply appearance-none w-5 h-5 rounded cursor-pointer;
	}
	.range::-moz-range-thumb {
		@apply appearance-none w-5 h-5 rounded cursor-pointer;
	}

	.input-slider[theme="default"] {
		.range {
			@apply bg-white dark:bg-gray-800 shadow h-3 transition hover:shadow-md;
		}
		.range::-webkit-slider-thumb {
			@apply bg-gradient-to-b from-yellow-400 to-yellow-500 dark:from-red-500 dark:to-red-600 shadow;
		}
		.range::-moz-range-thumb {
			@apply bg-gradient-to-b from-yellow-400 to-yellow-500 shadow;
		}
		.value {
			@apply bg-gray-100 dark:bg-gray-600 font-semibold;
		}
	}
</style>
