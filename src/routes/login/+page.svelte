
<script>
  let email = $state('');
  let password = $state('');
  let showPassword = $state(false);
  let error = $state('');

  function handleLogin(event) {
    event.preventDefault();
    error = '';

    if (!email || !password) {
      error = 'Please fill in all fields.';
      return;
    }

    error = 'Authentication is not connected yet. Supabase integration is coming next.';
  }
</script>

<svelte:head>
  <title>Log In | DevSpace</title>
</svelte:head>

<main class="auth-page">
  <section class="auth-card">
    <a href="/" class="logo">Dev<span>Space</span></a>

    <div class="heading">
      <h1>Welcome back</h1>
      <p>Log in to continue exploring and sharing projects.</p>
    </div>

    <form onsubmit={handleLogin}>
      <label for="email">Email address</label>
      <input
        id="email"
        type="email"
        placeholder="you@example.com"
        bind:value={email}
        required
      />

      <label for="password">Password</label>
      <div class="password-field">
        <input
          id="password"
          type={showPassword ? 'text' : 'password'}
          placeholder="Enter your password"
          bind:value={password}
          required
        />

        <button type="button" onclick={() => showPassword = !showPassword}>
          {showPassword ? 'Hide' : 'Show'}
        </button>
      </div>

      <button type="submit" class="submit-button">Log In</button>

      {#if error}
        <p class="error-message">{error}</p>
      {/if}
    </form>

    <p class="switch-text">
      Don't have an account?
      <a href="/signup">Sign up</a>
    </p>
  </section>
</main>

<style>
  .auth-page {
    min-height: 75vh;
    display: grid;
    place-items: center;
    padding: 60px 20px;
    background: #f7f9fc;
  }

  .auth-card {
    width: 100%;
    max-width: 440px;
    padding: 40px;
    background: white;
    border: 1px solid #e8edf4;
    border-radius: 16px;
  }

  .logo {
    font-size: 25px;
    font-weight: 800;
    color: #26364a;
    text-decoration: none;
  }

  .logo span {
    color: #5288d9;
  }

  .heading {
    margin: 35px 0 28px;
  }

  h1 {
    color: #26364a;
    font-size: 29px;
    margin-bottom: 10px;
  }

  .heading p,
  .switch-text {
    color: #778397;
    font-size: 14px;
    line-height: 1.7;
  }

  form {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  label {
    color: #26364a;
    font-size: 13px;
    font-weight: 600;
  }

  input {
    width: 100%;
    box-sizing: border-box;
    padding: 13px;
    border: 1px solid #dfe6ef;
    border-radius: 8px;
    outline: none;
    font-size: 14px;
    margin-bottom: 10px;
  }

  input:focus {
    border-color: #5288d9;
  }

  .password-field {
    display: flex;
    align-items: center;
    position: relative;
  }

  .password-field input {
    padding-right: 65px;
  }

  .password-field button {
    position: absolute;
    right: 12px;
    top: 12px;
    border: none;
    background: transparent;
    color: #5288d9;
    cursor: pointer;
  }

  .submit-button {
    padding: 14px;
    background: #5288d9;
    color: white;
    border: none;
    border-radius: 8px;
    font-weight: 600;
    cursor: pointer;
    margin-top: 8px;
  }

  .submit-button:hover {
    background: #3f73c0;
  }

  .switch-text {
    text-align: center;
    margin-top: 25px;
  }

  .switch-text a {
    color: #5288d9;
    font-weight: 600;
    text-decoration: none;
  }

  .error-message {
    color: #c0392b;
    font-size: 13px;
    line-height: 1.5;
  }

  @media (max-width: 480px) {
    .auth-card {
      padding: 28px 22px;
    }
  }
</style>
