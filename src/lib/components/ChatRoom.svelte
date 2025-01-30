<script lang="ts">
  import { page } from '$app/state';
  import { onMount, onDestroy } from 'svelte';

  let { data: { session, supabase } } = $derived(page);
  let messages = $state<any[]>([]);
  let newMessage = $state('');
  let messagesContainer: HTMLDivElement;

  function scrollToBottom() {
    if (messagesContainer) {
      messagesContainer.scrollTop = messagesContainer.scrollHeight;
    }
  }

  const subscription = supabase
    .channel('messages')
    .on(
      'postgres_changes',
      {
        event: '*',
        schema: 'public',
        table: 'messages'
      },
      (payload) => {
        if (payload.eventType === 'INSERT') {
          messages = [...messages, payload.new];
          // Scroll to bottom when new message arrives
          setTimeout(scrollToBottom, 0);
        }
      }
    )
    .subscribe((status) => {
      console.log('Subscription status:', status);
    });

  async function loadMessages() {
    const { data } = await supabase
      .from('messages')
      .select('*')
      .order('created_at', { ascending: true });
    messages = data ?? [];
    // Scroll to bottom after messages load
    setTimeout(scrollToBottom, 0);
  }

  async function sendMessage() {
    if (!session || !newMessage.trim()) return;

    const { data, error } = await supabase
      .from('messages')
      .insert({
        message: newMessage.trim(),
        user_id: session.user.id,
        username: session.user.email?.split('@')[0]
      })
      .select();

    if (error) console.error(error);
    else {
      newMessage = '';
    }
  }

  onMount(() => {
    loadMessages();
  });

  onDestroy(() => {
    subscription.unsubscribe();
  });
</script>

<div class="max-w-2xl mx-auto p-4">
  <div class="bg-background shadow-lg rounded-lg">
    <!-- Messages container -->
    <div 
      bind:this={messagesContainer}
      class="h-96 overflow-y-auto p-4 space-y-4"
    >
      {#each messages as message}
        <div class={`flex ${message.user_id === session?.user.id ? 'justify-end' : 'justify-start'}`}>
          <div class={`max-w-[70%] rounded-lg px-4 py-2 ${
            message.user_id === session?.user.id 
              ? 'bg-secondary text-text' 
              : 'bg-secondary/50 text-text-secondary'
          }`}>
            <div class="text-sm font-semibold">{message.username}</div>
            <div class="break-words">{message.message}</div>
          </div>
        </div>
      {/each}
    </div>

    <!-- Input area -->
    {#if session}
      <div class="border-t border-secondary p-4">
        <form 
          class="flex gap-2" 
          onsubmit={(e) => {
            e.preventDefault();
            sendMessage();
          }}
        >
          <input
            type="text"
            bind:value={newMessage}
            placeholder="Type a message..."
            class="flex-1 rounded-md px-3 py-2 bg-background border border-secondary text-text focus:outline-none focus:ring-2 focus:ring-secondary"
          />
          <button
            type="submit"
            class="px-4 py-2 rounded-md text-sm font-medium text-text-secondary bg-secondary hover:bg-secondary/80"
          >
            Send
          </button>
        </form>
      </div>
    {:else}
      <div class="p-4 text-center text-text-secondary">
        Please log in to participate in the chat
      </div>
    {/if}
  </div>
</div>
