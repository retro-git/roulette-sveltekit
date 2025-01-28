<script lang="ts">
  import { page } from '$app/state';

  type NavItem = {
    href: string;
    label: string;
  };

  let isOpen = $state(false);
  
  const navItems: NavItem[] = [
    { href: '/', label: 'Home' },
    { href: '/double', label: 'Double' }
  ];

  let currentPath = $derived(page.url.pathname);
</script>

<nav class="bg-white shadow-lg">
  <div class="max-w-7xl mx-auto px-4">
    <div class="flex justify-between h-16">
      <div class="flex items-center">
        <a href="/" class="flex items-center">
          <span class="text-xl font-bold text-gray-800">MyApp</span>
        </a>
      </div>

      <!-- Desktop Navigation -->
      <div class="hidden sm:flex sm:items-center">
        {#each navItems as item}
          <a
            href={item.href}
            class={`px-3 py-2 rounded-md text-sm font-medium ${
              currentPath === item.href 
                ? 'bg-gray-100 text-gray-900' 
                : 'text-gray-600 hover:bg-gray-50 hover:text-gray-900'
            }`}
          >
            {item.label}
          </a>
        {/each}
      </div>

      <!-- Mobile menu button -->
      <div class="flex items-center sm:hidden">
        <button
          onclick={() => isOpen = !isOpen}
          class="inline-flex items-center justify-center p-2 rounded-md text-gray-600 
            hover:text-gray-900 hover:bg-gray-100 focus:outline-none"
          aria-expanded={isOpen}
        >
          <span class="sr-only">{isOpen ? 'Close menu' : 'Open menu'}</span>
          <svg
            class="h-6 w-6"
            stroke="currentColor"
            fill="none"
            viewBox="0 0 24 24"
          >
            {#if isOpen}
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              />
            {:else}
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M4 6h16M4 12h16M4 18h16"
              />
            {/if}
          </svg>
        </button>
      </div>
    </div>

    <!-- Mobile menu -->
    {#if isOpen}
      <div class="sm:hidden pb-3 pt-2">
        {#each navItems as item}
          <a
            href={item.href}
            class={`block px-3 py-2 rounded-md text-base font-medium ${
              currentPath === item.href
                ? 'bg-gray-100 text-gray-900'
                : 'text-gray-600 hover:bg-gray-50 hover:text-gray-900'
            }`}
          >
            {item.label}
          </a>
        {/each}
      </div>
    {/if}
  </div>
</nav>