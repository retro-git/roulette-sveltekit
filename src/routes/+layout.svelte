<script lang="ts">
	import "../app.css"; // Tailwind CSS
	import Navbar from "$lib/components/Navbar.svelte";

	let { children } = $props();
</script>

<div class="min-h-screen bg-[var(--color-background)] text-[var(--color-text)]">
	<Navbar />

	<main class="max-w-[var(--container-max)] mx-auto px-4 py-8">
		{@render children()}
	</main>
</div>
