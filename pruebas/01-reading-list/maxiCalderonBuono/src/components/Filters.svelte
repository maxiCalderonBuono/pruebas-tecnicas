<script lang="ts">
	import type { LibraryElement } from '../types';
	import Tab from './Tab.svelte';
	import { createEventDispatcher } from 'svelte';
	import { NO_FILTER_APPLIED } from '$lib/const/filters';
	import { getUniqueGenres } from '$lib/utils/filters';

	const dispatch = createEventDispatcher();

	export let library: LibraryElement[];
	export let availables: number;
	export let savedFilter: string;

	let uniqueFilters = getUniqueGenres(NO_FILTER_APPLIED, library);
	let activeFilter: string = savedFilter;

	function handleInternalState(e: CustomEvent<{ filter: string }>) {
		const selectedFilter = e.detail.filter;
		activeFilter = selectedFilter;
		dispatch('selectedfilter', {
			filter: selectedFilter
		});
	}
</script>

{#each uniqueFilters as filter}
	<Tab
		{filter}
		on:selectedfilter={handleInternalState}
		{availables}
		active={activeFilter === filter}
	/>
{/each}
