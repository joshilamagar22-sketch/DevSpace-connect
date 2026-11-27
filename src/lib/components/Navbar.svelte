
<script>
  let menuOpen = $state(false);

  const links = [
    { name: 'Home', href: '/' },
    { name: 'Explore', href: '/explore' },
    { name: 'My Repositories', href: '/my-repositories' },
    { name: 'Community', href: '/community' },
    { name: 'About', href: '/about' }
  ];
</script>

<header class="navbar">
  <a href="/" class="logo">Dev<span>Space</span></a>

  <button
    class="menu-toggle"
    onclick={() => menuOpen = !menuOpen}
    aria-label="Toggle navigation"
    aria-expanded={menuOpen}
  >
    {menuOpen ? '✕' : '☰'}
  </button>

  <nav class:open={menuOpen}>
    {#each links as link}
      <a href={link.href} onclick={() => menuOpen = false}>
        {link.name}
      </a>
    {/each}
  </nav>

  <div class="auth-buttons">
    <a href="/login" class="login">Log in</a>
    <a href="/signup" class="signup">Sign up</a>
  </div>
</header>

<style>
  .navbar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 20px 6%;
    background: #ffffff;
    border-bottom: 1px solid #e9edf3;
    position: relative;
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

  nav {
    display: flex;
    gap: 25px;
  }

  nav a,
  .login {
    color: #526174;
    text-decoration: none;
    font-size: 14px;
    font-weight: 500;
  }

  nav a:hover,
  .login:hover {
    color: #5288d9;
  }

  .auth-buttons {
    display: flex;
    align-items: center;
    gap: 18px;
  }

  .signup {
    background: #5288d9;
    color: white;
    padding: 10px 18px;
    border-radius: 8px;
    text-decoration: none;
    font-size: 14px;
  }

  .menu-toggle {
    display: none;
    border: none;
    background: transparent;
    font-size: 24px;
    cursor: pointer;
  }

  @media (max-width: 900px) {
    .menu-toggle {
      display: block;
    }

    nav {
      display: none;
      position: absolute;
      top: 100%;
      left: 0;
      width: 100%;
      background: white;
      flex-direction: column;
      padding: 20px 6%;
      gap: 20px;
      z-index: 10;
      border-bottom: 1px solid #e9edf3;
    }

    nav.open {
      display: flex;
    }

    .auth-buttons {
      gap: 10px;
    }
  }
</style>
