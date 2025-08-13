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

<form class="grid w-full max-w-sm content-center">
	<fieldset class="fieldset">
		<legend class="fieldset-legend">Username</legend>
		<input type="text" name="email" bind:value={email} class="input w-full" />
	</fieldset>

	<fieldset class="fieldset">
		<legend class="fieldset-legend">Password</legend>
		<input type="password" name="password" bind:value={password} class="input w-full" />
	</fieldset>
	<button class="btn mt-4 btn-primary" onclick={() => submitHandler()}>Login</button>
</form>
