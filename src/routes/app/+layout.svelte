<script lang="ts">
	import ArkToaster from '$lib/ui/ArkToaster.svelte';

	let { children } = $props();
</script>

<ArkToaster />
{@render children()}
