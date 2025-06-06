<script lang="ts">
  import { goto, invalidate } from "$app/navigation";
  import { onMount } from "svelte";
  import "../app.css";
  let { data, children } = $props();

  /**
   * We use the $derived rune so that
   * `supabase` and `session` are updated
   * during invalidation. $state doesn't do this.
   *
   * An updated supabase client isn't typically needed,
   * but the ssr libary returns a cached client
   * for us during invalidation. Otherwise we'd be
   * initializing a client during every invalidation.
   */
  let { supabase, session } = $derived(data);

  onMount(() => {
    const {
      data: { subscription },
    } = supabase.auth.onAuthStateChange(async (event, _session) => {
      /**
       * Instead of invalidating, you could call
       * `session = _session` below and you wouldn't
       * necessarily need to call `invalidate`.
       */
      if (_session?.expires_at !== session?.expires_at) {
        /**
         * We typically only call `signOut()` on the server side,
         * but if `_session` is null - from the user
         * being deleted or the supabase client
         * failing to refresh a token, for example -
         * the SIGNED_OUT event is fired, and
         * calling `goto` ensures the user's screen
         * reflects that they're logged out.
         * Note that the invalidation still happens.
         */
        if (event === "SIGNED_OUT") await goto("/app");

        invalidate("supabase:auth");
      }
    });

    return () => subscription.unsubscribe();
  });
</script>

<!-- Navbar -->
<nav
  class="border-b-2 pb-2 pt-2 flex items-center justify-between px-4 bg-white shadow-sm"
>
  <div class="flex items-center space-x-4">
    <a href="/" class="text-blue-600 font-semibold hover:underline">Inicio</a>
    {#if session}
      <a href="/app" class="text-blue-600 font-semibold hover:underline"
        >Dashboard</a
      >
      <a href="/self" class="text-blue-600 font-semibold hover:underline"
        >Perfil</a
      >
    {/if}
  </div>
  <div class="flex items-center space-x-4">
    {#if session}
      <img
        class="w-8 h-8 rounded-full"
        src={session.user.user_metadata.avatar_url ??
          "https://api.dicebear.com/8.x/fun-emoji/svg"}
        alt="person_avatar"
      />
      <p class="text-sm text-gray-600">{session.user.user_metadata.nickname}</p>
      <p class="text-sm text-gray-600">
        Expira: {session?.expires_at
          ? new Date(session.expires_at * 1000).toLocaleString()
          : "unknown"}
      </p>
      <form method="POST" action="auth?/signout">
        <button class="bg-red-500 text-white px-3 py-1 rounded hover:bg-red-600"
          >Cerrar Sesion</button
        >
      </form>
    {:else}
      <a
        href="/auth"
        class="bg-blue-600 text-white px-3 py-1 rounded hover:bg-blue-700"
        >Iniciar Sesion</a
      >
    {/if}
  </div>
</nav>

{@render children?.()}
