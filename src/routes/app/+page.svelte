<script lang="ts">
	import { goto } from '$app/navigation';
	import { pb } from '$lib/pb';
	import { onMount } from 'svelte';
	import dayjs from 'dayjs';
	import { Calendar, DayGrid } from '@event-calendar/core';
	import utc from 'dayjs/plugin/utc';
	import timezone from 'dayjs/plugin/timezone';
	import PageWrapper from '$lib/PageWrapper.svelte';

	dayjs.extend(utc);
	dayjs.extend(timezone);

	let results: SprayDB[] | undefined = $state([]);

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

	let times = $derived.by(() => {
		let times: Calendar.EventInput[] = [];
		if (results && results.length > 0) {
			for (const r of results) {
				const n = dayjs.utc(r.time).tz('Asia/Singapore');
				times.push({ start: n.toDate(), end: n.toDate(), title: `— Sprayed` });
			}
		}
		return times;
	});

	let options: Calendar.Options = $derived.by(() => {
		return {
			view: 'dayGridMonth',
			events: [...times],
			selectBackgroundColor: 'red',
			eventBackgroundColor: '#4a4a7d'
			// eventContent: () => {
			// 	return `✔ Sprayed`;
			// }
		};
	});
</script>

<PageWrapper {pb}>
	<main class="grid w-full content-start justify-items-center gap-8">
		<h2 class="text-6xl font-semibold">Spray Logs</h2>
		<button class="btn btn-xl btn-primary" onclick={handleClick}>Just Sprayed!</button>
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
		<div class="w-full px-2 lg:px-12">
			{#key results}
				<Calendar plugins={[DayGrid]} {options} />
			{/key}
		</div>
	</main>
</PageWrapper>
<span class="ec ec-time-grid ec-title hidden"></span>

<style>
	:global {
		.ec-title {
			font-size: 1.5rem;
			font-weight: 700 !important;
		}

		.ec-today {
			background-color: var(--color-primary-content) !important;
		}

		/* .ec-event-title {
			margin-inline-start: 1em;
		} */
	}
</style>
