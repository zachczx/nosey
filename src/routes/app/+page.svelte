<script lang="ts">
	import { goto } from '$app/navigation';
	import { pb } from '$lib/pb';
	import { onMount } from 'svelte';
	import dayjs from 'dayjs';
	import { Calendar, DayGrid } from '@event-calendar/core';

	let results: Promise<SprayDB[]> = $state(new Promise(() => []));
	let times = $derived.by(() => {
		let times: string[] = [];
		if (results && results.length > 0) {
			for (const r of results) {
				times.push(r.created);
			}
		}
	});
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

	async function handleClick() {
		const resp = await pb.collection('spray').create({
			user: pb.authStore.record?.id
		});

		console.log(resp);

		results = (async () => {
			const results: SprayDB[] = await pb.collection('spray').getFullList({
				sort: '-created'
			});

			return results;
		})();
	}

	let options: Calendar.Options = $derived.by(() => {
		return {
			view: 'dayGridMonth',
			events: []
			// dayHeaderFormat: (time: Date) => {
			// 	return dayjs(time).format('ddd').toString() + '\r\n' + dayjs(time).format('D').toString();
			// },
			// eventBackgroundColor: '#4a4a7d',
			// firstDay: 1,
			// slotLabelFormat: { hour: 'numeric' },
			// events: [],
			// headerToolbar: {
			// 	start: 'title',
			// 	center: '',
			// 	end: 'today prev,next'
			// }
			// titleFormat: (start: Date, end: Date) => {
			// 	const s = dayjs(start);
			// 	const e = dayjs(end);
			// 	currentMonthText = s.format('MMMM YYYY').toString();
			// 	if (s.get('month') === e.get('month')) {
			// 		return s.format('D').toString() + ' – ' + e.format('D MMM').toString();
			// 	}
			// 	return s.format('D MMM').toString() + ' – ' + e.format('D MMM').toString();
			// }
		};
	});
</script>

<main class="grid w-full xl:grid-cols-2">
	<div class="grid content-start justify-items-center gap-8">
		<h2 class="text-6xl font-semibold">Spray Logs</h2>
		<button class="btn btn-primary" onclick={() => handleClick()}>Add Spray!</button>
		{#await results}
			<!-- <div class="loader"></div> -->
		{:then results}
			<ul class="list-disc ps-6">
				{#each results as result}
					{@const formatted = dayjs(result.created).format('DD/MM/YY (ddd)')}
					<li>{formatted}</li>
				{/each}
			</ul>
		{/await}
	</div>
	<div class="grid w-full content-start justify-items-center gap-8">
		<h2 class="text-6xl font-semibold">Calendar</h2>
		<div class="w-full px-12"><Calendar plugins={[DayGrid]} {options} /></div>
	</div>
</main>
<span class="ec ec-time-grid"></span>
