<script lang="ts">
	import { goto } from '$app/navigation';
	import { pb } from '$lib/pb';
	import { onMount } from 'svelte';
	import dayjs from 'dayjs';

	let results: Promise<SprayDB[]> = $state(new Promise(() => []));
	onMount(async () => {
		if (!pb.authStore.isValid) {
			goto('/login');
		}

		if (pb.authStore.isValid) {
			results = (async () => {
				const results: SprayDB[] = await pb.collection('spray').getFullList({
					sort: '-created'
				});

				return results;
			})();
		}
	});
</script>

<main class="grid w-full xl:grid-cols-2">
	<div class="grid content-center justify-items-center gap-8">
		<h2 class="text-6xl font-semibold">Spray Logs</h2>

		{#await results}
			<div class="loader"></div>
		{:then results}
			<ul class="list-disc ps-6">
				{#each results as result}
					{@const formatted = dayjs(result.created).format('DD/MM/YY')}
					<li>{formatted}</li>
				{/each}
			</ul>
		{/await}
	</div>
	<div class="grid content-center justify-items-center gap-8">
		<h2 class="text-6xl font-semibold">Calendar</h2>
	</div>
</main>
