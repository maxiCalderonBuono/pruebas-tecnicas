<script lang="ts">
	import { goto } from '$app/navigation';
	import { fly } from 'svelte/transition';
	import Icon from '@iconify/svelte';
	import api from '$lib/services/api';

	interface searchResults {
		title: string;
		desc: string;
		slug: string;
	}

	let titles: searchResults[];

	api.books
		.list()
		.then((data) =>
			data.map((book) => {
				return {
					title: book.title,
					desc: book.synopsis,
					slug: book.title
				};
			})
		)
		.then((results) => (titles = results))
		.catch((err) => console.error('Error al leer los datos', err));

	let searchValue: string;

	let filteredTitles: searchResults[] = [];

	function filterTitles() {
		let storageTitles: searchResults[] = [];

		if (searchValue) {
			titles.forEach((title) => {
				if (title.title.toLowerCase().trim().startsWith(searchValue.trim().toLowerCase())) {
					storageTitles = [
						...storageTitles,
						{ title: makeMatchBold(title.title), desc: title.desc, slug: title.slug }
					];
				}
			});
		} else {
			filteredTitles = [];
		}

		filteredTitles = storageTitles;
	}

	export let label: String;
	export let id: string;
	export let action: string;
	export let placeholder: string;

	let highlighted: number = -1;

	function submitSearch() {
		if (filteredTitles.length > 0) {
			const selectedTitle = filteredTitles[highlighted];
			searchValue = selectedTitle?.slug.toLowerCase();
			goto('/search/' + searchValue);
		} else if (searchValue) {
			const trimmedSearchValue = searchValue.trim().toLowerCase();
			const foundTitle = filteredTitles.find((title) => title.slug === trimmedSearchValue);
			if (foundTitle) {
				goto('/search/' + searchValue.toLowerCase());
			} else {
				searchValue = '';
			}
		} else {
			searchValue = '';
		}
	}

	const navigateList = (e: KeyboardEvent) => {
		if (e.key === 'ArrowDown' && highlighted < filteredTitles.length - 1) {
			highlighted === -1 ? (highlighted = 0) : (highlighted += 1);
		} else if (e.key === 'ArrowUp' && highlighted !== null) {
			highlighted === 0 ? (highlighted = filteredTitles.length - 1) : (highlighted -= 1);
		} else if (e.key === 'Enter') {
			console.log(filteredTitles[highlighted].slug.toLowerCase());
			searchValue = filteredTitles[highlighted].slug.toLowerCase();
		} else if (!searchValue) {
			highlighted = -1;
		} else {
			return;
		}
	};

	const makeMatchBold = (str: string) => {
		let matched = str.substring(0, searchValue.length);
		let makeBold = `<strong>${matched}</strong>`;
		let boldedMatch = str.replace(matched, makeBold);
		return boldedMatch;
	};
</script>

<svelte:window on:keydown={(e) => navigateList(e)} on:mousemove />
<form
	class="relative flex flex-col max-w-xl w-full"
	autocomplete="off"
	on:submit|preventDefault={submitSearch}
>
	<label class="font-bold text-2xl hidden" aria-label="Search bar" for={id}>{label}</label>

	<div class="flex h-12">
		<input
			class="input w-full
		rounded-lg h-full border-2 border-green-500
		p-2 outline-none"
			{placeholder}
			{id}
			type="text"
			bind:value={searchValue}
			on:input={filterTitles}
		/>
		{#if searchValue}
			<button
				out:fly={{ x: 0, duration: 500 }}
				in:fly={{ x: 50, duration: 500 }}
				class="btn variant-filled border-2 border-green-500 border-l-none font-bold h-12 absolute right-0 hover:bg-green-500 hover:text-gray-200 rounded-r-lg rounded-l-none"
				>{action}</button
			>
		{/if}
	</div>

	<dl class="absolute w-full top-11 mt-2 text-black z-20">
		{#each filteredTitles as option, i}
			<a
				href={`/search/${option.slug.toLocaleLowerCase()}`}
				out:fly={{ x: 0, duration: 500 }}
				in:fly={{ y: 50, duration: 500 }}
				class={`flex w-full items-center gap-3 ${
					highlighted === i ? 'bg-green-300' : 'bg-gray-200'
				} rounded-lg px-2 mb-2 hover:bg-lime-200 hover:cursor-pointer`}
			>
				<span class="rounded-full bg-indigo-400 p-3"><Icon icon="ion:book-outline" /></span>
				<span class="flex-auto overflow-hidden">
					<dt>{@html option.title}</dt>
					<dd class="truncate">{option.desc}</dd>
				</span>
			</a>
		{/each}
	</dl>
</form>
