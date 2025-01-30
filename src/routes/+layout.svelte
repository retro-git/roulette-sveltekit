<script lang="ts">
	import "../app.css"; // Tailwind CSS
	import Navbar from "$lib/components/Navbar.svelte";
	import { invalidate } from '$app/navigation';
	import { onMount } from 'svelte';

	let { data, children } = $props();
	let { session, supabase } = $derived(data);

	onMount(() => {
		const { data } = supabase.auth.onAuthStateChange((_, newSession) => {
			if (newSession?.expires_at !== session?.expires_at) {
				invalidate('supabase:auth');
			}
		});

		return () => data.subscription.unsubscribe();
	});
</script>

<div class="min-h-screen bg-background text-text">
	<Navbar />

	<main class="max-w-max mx-auto px-4 py-8">
		{@render children()}
	</main>
</div>
