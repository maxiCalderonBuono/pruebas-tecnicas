<script lang="ts">
	import { RangeSlider } from '@skeletonlabs/skeleton';
	import type { LibraryElement } from '../types';
	import { createEventDispatcher } from 'svelte';
	import { getRangeOfPages } from '$lib/utils/filters';

	const dispatch = createEventDispatcher();

	export let library: LibraryElement[];

	const { maxPage, minPage } = getRangeOfPages(library);

	let value: number = maxPage;

	$: dispatch('change', { value });
</script>

<RangeSlider
	name="range-slider"
	bind:value
	max={maxPage}
	min={minPage}
	step={1}
	class="w-5/6 md:w-2/3 lg:w-2/5 mx-auto"
>
	<div class="flex justify-between items-center">
		<div class="font-bold text-xl">Pages:</div>
		<div class=" text-lime-200 font-bold text-xl">{value}</div>
	</div>
</RangeSlider>
