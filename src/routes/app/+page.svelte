<script lang="ts">
	import { goto } from '$app/navigation';
	import { pb } from '$lib/pb';
	import { onMount } from 'svelte';
	import dayjs from 'dayjs';
	import { Calendar, DayGrid, Interaction } from '@event-calendar/core';
	import utc from 'dayjs/plugin/utc';
	import timezone from 'dayjs/plugin/timezone';
	import PageWrapper from '$lib/PageWrapper.svelte';
	import { addToast } from '$lib/ui/ArkToaster.svelte';
	import MaterialSymbolsCheck from '$lib/assets/svg/MaterialSymbolsCheck.svelte';
	import MaterialSymbolsAdd from '$lib/assets/svg/MaterialSymbolsAdd.svelte';

	dayjs.extend(utc);
	dayjs.extend(timezone);

	let results: SprayDB[] | undefined = $state([]);
	let singleDay: SprayDB[] | undefined = $state([]);

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
		const result = await pb.collection('spray').create({
			user: pb.authStore.record?.id,
			time: dayjs.tz(new Date(), 'Asia/Singapore')
		});

		if (result.id) {
			addToast('success', 'Added successfully!');
		}

		results = await pb.collection('spray').getFullList({
			sort: '-created'
		});
	}

	let times = $derived.by(() => {
		let times: Calendar.EventInput[] = [];
		if (results && results.length > 0) {
			for (const r of results) {
				//Adding the timezone here creates a problem for mobile devices, not sure why => .tz('Asia/Singapore');
				const n = dayjs.utc(r.time);
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
			eventBackgroundColor: '#4a4a7d',
			dateClick: async (info) => {
				singleDay = results?.filter((day) => {
					return dayjs(day.time).get('date') == dayjs(info.date).get('date');
				});
				singleDayModal.showModal();
			}
		};
	});

	let singleDayModal = $state() as HTMLDialogElement;
</script>

<PageWrapper {pb}>
	<main class="grid w-full content-start justify-items-center gap-8">
		<h2 class="text-6xl font-semibold">Spray Logs</h2>
		<button
			class="btn flex items-center gap-2 btn-xl btn-primary max-md:w-full"
			onclick={handleClick}><MaterialSymbolsAdd class="size-[1.3em]" />Add Spray</button
		>
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
				<Calendar plugins={[DayGrid, Interaction]} {options} />
			{/key}
		</div>
	</main>
</PageWrapper>
<span class="ec ec-time-grid ec-title hidden"></span>

<dialog bind:this={singleDayModal} class="modal">
	<div class="modal-box">
		{#if singleDay && singleDay.length > 0}
			{@const theDay = dayjs(singleDay[0].time).format('DD MMM YYYY')}
			<h3 class="text-lg font-bold">{theDay}</h3>
			<div class="py-4">
				<ul class="list-disc">
					{#each singleDay as day}
						{@const formatted = dayjs(day.time).format('h:mma')}
						<li class="flex items-center gap-2">
							<MaterialSymbolsCheck class="size-[1.3em] text-success" />
							{formatted}
						</li>
					{/each}
				</ul>
			</div>
		{:else}
			<h3 class="text-lg font-bold">Nothing here!</h3>
		{/if}
		<div class="modal-action">
			<form method="dialog">
				<button class="btn">Close</button>
			</form>
		</div>
	</div>
</dialog>

<style>
	:global {
		.ec-title {
			font-size: 1.5rem;
			font-weight: 700 !important;
		}

		.ec-today {
			background-color: var(--color-primary-content) !important;
		}

		.ec-day {
			&:hover {
				background-color: rgba(242, 189, 205, 0.3);
				.ec-day-head {
					font-weight: 600 !important;
				}
				color: var(--color-primary);
			}
		}

		.ec-bg-events {
			&:hover {
				cursor: pointer;
			}
		}

		/* .ec-event-title {
			margin-inline-start: 1em;
		} */
	}
</style>
