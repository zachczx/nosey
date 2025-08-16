<script>
	let { pb, children, title } = $props();
</script>

<svelte:head>
	<title>{title}</title>
</svelte:head>

<div
	class="grid min-h-dvh w-full grid-rows-[auto_1fr] content-start justify-items-center p-1 lg:p-4"
>
	<div class="flex w-full justify-end px-2 lg:px-12">
		{#if pb.authStore.isValid}
			<a href="/logout" class="underline">Logout</a>
		{:else}
			<a href="/register" class="underline">Register</a>&nbsp;&nbsp;|&nbsp;&nbsp;<a
				href="/login"
				class="underline">Login</a
			>
		{/if}
	</div>
	{@render children()}
</div>
