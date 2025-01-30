<script lang="ts">
  // Define available OAuth providers
  const oauthProviders = [
    {
      name: 'Discord',
      action: '?/discord',
      icon: '🎮' // You can replace this with an actual Discord icon
    }
    // Add more providers here later, for example:
    // { name: 'Google', action: '?/google', icon: '🔍' }
    // { name: 'GitHub', action: '?/github', icon: '📝' }
  ]
</script>

<form method="POST" action="?/login">
    <label>
      Email
      <input name="email" type="email" />
    </label>
    <label>
      Password
      <input name="password" type="password" />
    </label>
    <button>Login</button>
    <button formaction="?/signup">Sign up</button>
  </form>

<div class="oauth-section">
  <p>Or continue with:</p>
  <div class="oauth-buttons">
    {#each oauthProviders as provider}
      <form method="POST" action={provider.action}>
        <button type="submit">
          <span class="icon">{provider.icon}</span>
          {provider.name}
        </button>
      </form>
    {/each}
  </div>
</div>

<style>
  .oauth-section {
    margin-top: 1rem;
    text-align: center;
  }

  .oauth-buttons {
    display: flex;
    gap: 1rem;
    justify-content: center;
  }

  .icon {
    margin-right: 0.5rem;
  }
</style>