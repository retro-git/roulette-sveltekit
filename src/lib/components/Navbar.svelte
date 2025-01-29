<script lang="ts">
  import { onMount } from 'svelte';
  import { page } from '$app/state';

  type NavItem = {
    href: string;
    label: string;
  };

  let isOpen = $state(false);
  let isDark = $state(false);
  
  const navItems: NavItem[] = [
    { href: '/', label: 'Home' },
    { href: '/double', label: 'Double' }
  ];

  let currentPath = $derived(page.url.pathname);

  function setDarkMode(value: boolean) {
    isDark = value;
    if (value) {
      document.documentElement.classList.add('dark');
    } else {
      document.documentElement.classList.remove('dark');
    }
    localStorage.setItem('theme', value ? 'dark' : 'light');
  }

  function toggleDarkMode() {
    setDarkMode(!isDark);
  }

  onMount(() => {
    // Check localStorage first
    const savedTheme = localStorage.getItem('theme');
    
    if (savedTheme) {
      // Use saved preference if it exists
      setDarkMode(savedTheme === 'dark');
    } else {
      // Otherwise check system preference
      const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
      setDarkMode(mediaQuery.matches);

      // Listen for system preference changes
      mediaQuery.addEventListener('change', (e) => {
        setDarkMode(e.matches);
      });
    }
  });
</script>

<nav class="bg-white dark:bg-gray-800 shadow-lg">
  <div class="max-w-7xl mx-auto px-4">
    <div class="flex justify-between h-16">
      <div class="flex items-center">
        <a href="/" class="flex items-center">
          <span class="text-xl font-bold text-gray-800 dark:text-white">MyApp</span>
        </a>
      </div>

      <!-- Desktop Navigation -->
      <div class="hidden sm:flex sm:items-center">
        {#each navItems as item}
          <a
            href={item.href}
            class={`px-3 py-2 rounded-md text-sm font-medium ${
              currentPath === item.href 
                ? 'bg-gray-100 text-gray-900 dark:bg-gray-700 dark:text-white' 
                : 'text-gray-600 hover:bg-gray-50 hover:text-gray-900 dark:text-gray-300 dark:hover:bg-gray-700 dark:hover:text-white'
            }`}
          >
            {item.label}
          </a>
        {/each}
        
        <!-- Dark mode toggle button -->
        <button
          onclick={toggleDarkMode}
          class="ml-4 p-2 rounded-md text-gray-600 hover:text-gray-900 hover:bg-gray-100 
            dark:text-gray-300 dark:hover:text-white dark:hover:bg-gray-700 focus:outline-none"
          aria-label="Toggle dark mode"
        >
          {#if isDark}
            <!-- Sun icon -->
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
                d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"
              />
            </svg>
          {:else}
            <!-- Moon icon -->
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
                d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"
              />
            </svg>
          {/if}
        </button>
      </div>

      <!-- Mobile menu button -->
      <div class="flex items-center sm:hidden">
        <button
          onclick={() => isOpen = !isOpen}
          class="inline-flex items-center justify-center p-2 rounded-md text-gray-600 
            hover:text-gray-900 hover:bg-gray-100 dark:text-gray-300 dark:hover:text-white dark:hover:bg-gray-700 focus:outline-none"
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
                ? 'bg-gray-100 text-gray-900 dark:bg-gray-700 dark:text-white'
                : 'text-gray-600 hover:bg-gray-50 hover:text-gray-900 dark:text-gray-300 dark:hover:bg-gray-700 dark:hover:text-white'
            }`}
          >
            {item.label}
          </a>
        {/each}
        
        <!-- Dark mode toggle button (mobile) -->
        <button
          onclick={toggleDarkMode}
          class="w-full text-left px-3 py-2 rounded-md text-base font-medium text-gray-600 
            hover:bg-gray-50 hover:text-gray-900 dark:text-gray-300 dark:hover:bg-gray-700 
            dark:hover:text-white"
        >
          <div class="flex items-center">
            {#if isDark}
              <svg class="w-5 h-5 mr-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
                  d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"
                />
              </svg>
            {:else}
              <svg class="w-5 h-5 mr-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
                  d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"
                />
              </svg>
            {/if}
            {isDark ? 'Light Mode' : 'Dark Mode'}
          </div>
        </button>
      </div>
    {/if}
  </div>
</nav>