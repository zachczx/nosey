<script lang="ts">
	import '../app.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
</svelte:head>

<div class="grid min-h-dvh w-full content-start justify-items-center p-1 lg:p-4">
	{@render children?.()}
</div>
