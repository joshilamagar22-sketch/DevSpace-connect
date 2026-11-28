
<script>
  import { supabase } from '../../lib/supabaseClient.js';

  import { page } from '$app/state';
  let name = $state('');
  let email = $state('');
  let password = $state('');
  let confirmPassword = $state('');
  let showPassword = $state(false);
  let message = $state('');
  let error = $state('');
  let loading = $state(false);

  /** @param {SubmitEvent} event */
  async function handleSignup(event) {
    event.preventDefault();

    error = '';
    message = '';

    if (password.length < 8) {
      error = 'Password must contain at least 8 characters.';
      return;
    }

    if (password !== confirmPassword) {
      error = 'Passwords do not match.';
      return;
    }

    loading = true;

    const { data, error: signupError } = await supabase.auth.signUp({
      email,
      password,
      options: {
        data: {
          full_name: name
        }
      }
    });

    loading = false;

    if (signupError) {
      error = signupError.message;
      return;
    }

    if (data.session) {
  const redirectTo = page.url.searchParams.get('redirect') || '/my-repositories';

  window.location.href = redirectTo;
} else {
  message = 'Account created! Please check your email to verify your account.';
}
  }
</script>

<svelte:head>
  <title>Sign Up | DevSpace</title>
</svelte:head>

<main class="auth-page">
  <section class="auth-card">
    <a href="/" class="logo">Dev<span>Space</span></a>

    <div class="heading">
      <h1>Create your account</h1>
      <p>Join DevSpace and start showcasing your projects.</p>
    </div>

    <form onsubmit={handleSignup}>
      <label for="name">Full name</label>
      <input id="name" type="text" placeholder="Your name" bind:value={name} required />

      <label for="email">Email address</label>
      <input id="email" type="email" placeholder="you@example.com" bind:value={email} required />

      <label for="password">Password</label>
      <div class="password-field">
        <input
          id="password"
          type={showPassword ? 'text' : 'password'}
          placeholder="At least 8 characters"
          bind:value={password}
          required
        />
        <button type="button" onclick={() => showPassword = !showPassword}>
          {showPassword ? 'Hide' : 'Show'}
        </button>
      </div>

      <label for="confirmPassword">Confirm password</label>
      <input
        id="confirmPassword"
        type="password"
        placeholder="Re-enter your password"
        bind:value={confirmPassword}
        required
      />

      <button type="submit" class="submit-button" disabled={loading}>
        {loading ? 'Creating account...' : 'Create Account'}
      </button>

      {#if error}
        <p class="error-message">{error}</p>
      {/if}

      {#if message}
        <p class="success-message">{message}</p>
      {/if}
    </form>

    <p class="switch-text">
      Already have an account?
      <a href="/login">Log in</a>
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
    margin: 30px 0 25px;
  }

  h1 {
    color: #26364a;
    font-size: 27px;
    margin-bottom: 10px;
  }

  .heading p, .switch-text {
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
    margin-bottom: 8px;
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
  }

  .submit-button:disabled {
    opacity: 0.6;
    cursor: not-allowed;
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
  }

  .success-message {
    color: #218653;
    font-size: 13px;
    line-height: 1.6;
  }

  @media (max-width: 480px) {
    .auth-card {
      padding: 28px 22px;
    }
  }
</style>
