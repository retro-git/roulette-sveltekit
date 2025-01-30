<script lang="ts">
    import type { LayoutData } from './$types'
    import { goto } from '$app/navigation'
    
    let { data, children } = $props<{data: LayoutData, children: any}>()
    let { supabase } = $derived(data)
  
    const logout = async () => {
      const { error } = await supabase.auth.signOut()
      if (error) {
        console.error(error)
      } else {
        await goto('/auth')
      }
    }
</script>

<style>
    button {
        cursor: pointer;
    }
</style>

<header>
  <nav>
    <a href="/">Home</a>
  </nav>
  <button onclick={logout}>Logout</button>
</header>
<main>
  {@render children()}
</main>