<script>
	import { goto } from '$app/navigation';
	import { pb } from '$lib/pb';

	if (pb.authStore.isValid) {
		goto('/app');
	}

	let email = $state('');
	let password = $state('');

	async function submitHandler() {
		try {
			const authData = await pb.collection('users').authWithPassword(email, password);
			console.log(authData);
			if (authData.token) {
				goto('/app');
			}
		} catch (err) {
			console.log(err);
		}
	}
</script>

<form class="grid w-full max-w-sm content-center gap-4">
	<input type="text" name="email" bind:value={email} class="input w-full" />
	<input type="password" name="password" bind:value={password} class="input w-full" />
	<button class="btn btn-primary" onclick={() => submitHandler()}>Submit</button>
</form>
