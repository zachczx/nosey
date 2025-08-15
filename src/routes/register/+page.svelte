<script>
	import { goto } from '$app/navigation';
	import PageWrapper from '$lib/PageWrapper.svelte';
	import { pb } from '$lib/pb';

	if (pb.authStore.isValid) {
		goto('/app');
	}

	let email = $state('');
	let password = $state('');

	async function submitHandler() {
		try {
			const data = {
				email: email,
				emailVisibility: true,
				password: password,
				passwordConfirm: password
			};

			const record = await pb.collection('users').create(data);

			console.log(record.id);
		} catch (err) {
			console.log(err);
		}

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

<PageWrapper {pb}>
	<form class="grid w-full max-w-sm content-center">
		<div class="mb-8">
			<h1 class="text-7xl font-medium text-primary">Nosey</h1>
			<h2 class="flex items-center gap-2 text-2xl">
				<svg
					xmlns="http://www.w3.org/2000/svg"
					width="1em"
					height="1em"
					class="material-symbols:subdirectory-arrow-right-rounded"
					viewBox="0 0 24 24"
					><!-- Icon from Material Symbols by Google - https://github.com/google/material-design-icons/blob/master/LICENSE --><path
						fill="currentColor"
						d="M13.3 20.275q-.3-.3-.3-.7t.3-.7L16.175 16H7q-.825 0-1.412-.587T5 14V5q0-.425.288-.712T6 4t.713.288T7 5v9h9.175l-2.9-2.9q-.3-.3-.288-.7t.288-.7q.3-.3.7-.312t.7.287L19.3 14.3q.15.15.212.325t.063.375t-.063.375t-.212.325l-4.575 4.575q-.3.3-.712.3t-.713-.3"
					/></svg
				>Register
			</h2>
		</div>
		<fieldset class="fieldset">
			<legend class="fieldset-legend">Username</legend>
			<input type="text" name="email" bind:value={email} class="input w-full" />
		</fieldset>

		<fieldset class="fieldset">
			<legend class="fieldset-legend">Password</legend>
			<input type="password" name="password" bind:value={password} class="input w-full" />
		</fieldset>
		<button class="btn mt-4 btn-primary" onclick={() => submitHandler()}>Register</button>
	</form>
</PageWrapper>
