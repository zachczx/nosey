<script lang="ts">
	import { goto } from '$app/navigation';
	import { pb } from '$lib/pb';

	if (pb.authStore.isValid) {
		goto('/app');
	} else {
		goto('/login');
	}
</script>
