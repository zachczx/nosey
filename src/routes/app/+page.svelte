<script lang="ts">
	import { goto } from '$app/navigation';
	import { pb } from '$lib/pb';
	import { onMount } from 'svelte';
	import dayjs from 'dayjs';
	import { Calendar, DayGrid } from '@event-calendar/core';
	import utc from 'dayjs/plugin/utc';
	import timezone from 'dayjs/plugin/timezone';

	dayjs.extend(utc);
	dayjs.extend(timezone);

	let results = $state([]);
	let times = $derived.by(() => {
		let times: string[] = [];
		if (results && results.length > 0) {
			for (const r of results) {
				console.log('before: ', r.time);
				const n = dayjs.utc(r.time).tz('Asia/Singapore');
				console.log('after: ', n.format());
				times.push(n);
			}
		}
		return times;
	});
	onMount(async () => {
		if (!pb.authStore.isValid) {
			goto('/login');
		}

		if (pb.authStore.isValid) {
			results = await pb.collection('spray').getFullList({
				sort: '-created'
			});
		}
	});

	async function handleClick() {
		const resp = await pb.collection('spray').create({
			user: pb.authStore.record?.id,
			time: dayjs.tz(new Date(), 'Asia/Singapore')
		});

		console.log(resp);

		results = await pb.collection('spray').getFullList({
			sort: '-created'
		});
	}

	let options: Calendar.Options = $derived.by(() => {
		return {
			view: 'dayGridMonth',
			events: [...times],

			eventBackgroundColor: '#4a4a7d',
			eventContent: () => {
				return `✔ Sprayed`;
			}
		};
	});
</script>

<main class="grid w-full content-start justify-items-center gap-8">
	<h2 class="text-6xl font-semibold">Spray Logs</h2>
	<button class="btn btn-lg btn-primary" onclick={() => handleClick()}>Just Sprayed!</button>
	<!-- {#await results}
			<div class="loader"></div>
		{:then results}
			<ul class="list-disc ps-6">
				{#each results as result}
					{@const formatted = dayjs(result.created).format('DD/MM/YY (ddd)')}
					<li>{formatted}</li>
				{/each}
			</ul>
		{/await} -->
	<div class="w-full px-2 lg:px-12"><Calendar plugins={[DayGrid]} {options} /></div>
</main>
<span class="ec ec-time-grid"></span>
