<script lang="ts">
  let { form } = $props();
</script>

<main class="min-h-screen flex flex-col items-center justify-center bg-gray-100 p-4">
  <div class="bg-white rounded-xl shadow-lg p-6 w-full max-w-md space-y-6">
    <h1 class="text-2xl font-bold text-center text-gray-800">Iniciar sesión</h1>

    <form method="POST" action="?/signin_email" class="space-y-4">
      <input name="email" placeholder="Email" type="email" value={form?.email ?? ""} class="w-full border px-3 py-2 rounded focus:outline-none focus:ring" />
      <input name="password" placeholder="Contraseña" type="password" class="w-full border px-3 py-2 rounded focus:outline-none focus:ring" />
      <button class="w-full bg-blue-600 text-white py-2 rounded hover:bg-blue-700">Login</button>
      <button formaction="?/signup" class="w-full bg-gray-200 text-gray-800 py-2 rounded hover:bg-gray-300">Registrarse</button>
    </form>

    <form method="POST" action="?/reset" class="space-y-2">
      <input name="email" placeholder="Email" type="email" class="w-full border px-3 py-2 rounded focus:outline-none focus:ring" />
      <button class="w-full bg-red-500 text-white py-2 rounded hover:bg-red-600">Restablecer contraseña</button>
    </form>

    {#if form?.message}
      <p class="text-green-600 text-sm text-center">{form.message}</p>
    {/if}
    {#if form?.error}
      <p class="text-red-600 text-sm text-center">{form.error}</p>
    {/if}

    {#if form?.verify}
      <form method="POST" action="?/verify_otp" class="space-y-2">
        <input name="otp" placeholder={`OTP enviado a ${form?.phone}`} type="text" class="w-full border px-3 py-2 rounded focus:outline-none focus:ring" />
        <input name="phone" type="hidden" value={form?.phone} />
        <button class="w-full bg-green-600 text-white py-2 rounded hover:bg-green-700">Verificar</button>
      </form>
    {/if}
  </div>
</main>
