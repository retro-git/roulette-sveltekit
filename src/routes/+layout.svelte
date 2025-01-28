<script lang="ts">
	import "../app.css"; // Tailwind CSS
	import Navbar from "$lib/components/Navbar.svelte";

	let { children } = $props();
</script>

<div class="min-h-screen bg-gray-50">
	<Navbar />

	<main class="max-w-7xl mx-auto px-4 py-8">
		{@render children()}
	</main>
</div>
