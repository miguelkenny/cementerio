<script lang="ts">
  let { data, form } = $props()
  let { session } = $derived(data)

  let has_email_provider = $state(false)

  $effect(() => {
    const providers = session?.user.app_metadata.providers
    has_email_provider = providers
      ? providers.some((p: string) => p === 'email')
      : session?.user.app_metadata.provider === 'email'
  })
</script>

{#if session}
  <div class="max-w-2xl mx-auto mt-10 space-y-8">
    <div class="bg-white rounded-2xl shadow p-6 space-y-2">
      <h2 class="text-2xl font-bold text-gray-800">👤 Tu perfil</h2>
      <p><strong>ID:</strong> {session.user.id}</p>
      <p><strong>Email:</strong> {session.user.email || "No definido"}</p>
      <p><strong>Teléfono:</strong> {session.user.phone || "No definido"}</p>
      <p><strong>Alias:</strong> {session.user.user_metadata.nickname || "No definido"}</p>
    </div>

    <div class="bg-white rounded-2xl shadow p-6 space-y-4">
      <h3 class="text-xl font-semibold text-gray-700">🔧 Opciones</h3>

      <form method="POST" action="?/delete_user" class="space-y-2">
        <label class="block font-medium">Eliminar usuario por ID:</label>
        <input name="user" type="text" class="input" />
        <button class="btn btn-danger">Eliminar</button>
      </form>

      <form method="POST" action="?/update_nickname" class="space-y-2">
        <label class="block font-medium">Cambiar alias:</label>
        <input name="nickname" type="text" class="input" value={form?.nickname ?? ""} />
        <div class="flex gap-2">
          <button class="btn btn-primary">Actualizar</button>
          <button formaction="?/delete_nickname" class="btn btn-secondary">Eliminar</button>
        </div>
      </form>

      <form method="POST" action="?/update_phone" class="space-y-2">
        <label class="block font-medium">Cambiar teléfono:</label>
        <input name="phone" type="text" class="input" />
        <button class="btn btn-primary">Actualizar</button>
      </form>

      {#if has_email_provider}
        <form method="POST" action="?/update_password" class="space-y-2">
          <label class="block font-medium">Cambiar contraseña:</label>
          <input name="password" type="password" class="input" />
          <button class="btn btn-primary">Cambiar</button>
        </form>
      {/if}

      {#if session.user.is_anonymous}
        <div class="space-y-2">
          <form method="POST" action="?/convert_provider">
            <button class="btn btn-primary w-full" name="provider" value="github">Convertir con GitHub</button>
          </form>
          <form method="POST" action="?/convert_email" class="space-y-2">
            <input name="email" type="email" class="input" placeholder="email" />
            <button class="btn btn-primary w-full">Convertir con email</button>
          </form>
        </div>
      {/if}
    </div>
  </div>
{/if}

{#if form?.message}
  <div class="max-w-2xl mx-auto mt-4 text-green-700 bg-green-100 p-3 rounded-md shadow">{form.message}</div>
{/if}

{#if form?.error}
  <div class="max-w-2xl mx-auto mt-4 text-red-700 bg-red-100 p-3 rounded-md shadow">{form.error}</div>
{/if}

{#if form?.verify}
  <div class="max-w-2xl mx-auto mt-6 bg-white rounded-2xl shadow p-6">
    <form method="POST" action="?/verify_otp" class="space-y-4">
      <input name="otp" placeholder={`OTP enviado a ${form?.phone ?? form?.email}`} class="input" />
      {#if form?.password_prompt}
        <input name="password" placeholder="Nueva contraseña" type="password" class="input" />
      {/if}
      {#if form?.phone}<input name="phone" type="hidden" value={form.phone}>{/if}
      {#if form?.email}<input name="email" type="hidden" value={form.email}>{/if}
      <button class="btn btn-primary w-full">Verificar</button>
    </form>
  </div>
{/if}

