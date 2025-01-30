<script lang="ts">
	import "../app.css"; // Tailwind CSS
	import Navbar from "$lib/components/Navbar.svelte";

	let { children } = $props();
</script>

<div class="min-h-screen bg-background text-text">
	<Navbar />

	<main class="max-w-max mx-auto px-4 py-8">
		{@render children()}
	</main>
</div>
