
<script>
  import { onMount } from 'svelte';
  import { supabase } from '../supabaseClient.js';

  let menuOpen = $state(false);
  let user = $state(null);
  let profile = $state(null);

  const links = [
    { name: 'Home', href: '/' },
    { name: 'Explore', href: '/explore' },
    { name: 'My Repositories', href: '/my-repositories' },
    { name: 'Saved Repositories', href: '/saved-repositories' },
    { name: 'Community', href: '/community' },
    { name: 'About', href: '/about' }
  ];

  let displayName = $derived(
    profile?.full_name || user?.user_metadata?.full_name || user?.email?.split('@')[0] || 'Developer'
  );

  let initials = $derived(
    displayName
      .split(' ')
      .filter(Boolean)
      .slice(0, 2)
      .map((word) => word[0].toUpperCase())
      .join('')
  );

  async function loadProfile(userId) {
    const { data, error } = await supabase
      .from('profiles')
      .select('full_name, username, avatar_url')
      .eq('id', userId)
      .maybeSingle();

    if (error) {
      console.error('Profile loading error:', error.message);
      profile = null;
      return;
    }

    profile = data;
  }

  onMount(async () => {
    const {
      data: { subscription }
    } = supabase.auth.onAuthStateChange((event, session) => {
      user = session?.user || null;

      if (session?.user) {
        void loadProfile(session.user.id);
      } else {
        profile = null;
      }
    });

    const {
      data: { session }
    } = await supabase.auth.getSession();

    user = session?.user || null;

    if (session?.user) {
      await loadProfile(session.user.id);
    }

    return () => subscription.unsubscribe();
  });

  async function logout() {
    const { error } = await supabase.auth.signOut();

    if (error) {
      console.error('Logout failed:', error.message);
      return;
    }

    window.location.href = '/';
  }
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
  {#if link.href !== '/saved-repositories' || user}
    <a href={link.href} onclick={() => menuOpen = false}>
      {link.name}
    </a>
  {/if}
{/each}
  </nav>

  <div class="auth-buttons">
    {#if user}
      <a href="/profile" class="profile-link">
        {#if profile?.avatar_url}
          <img
            src={profile.avatar_url}
            alt="Profile"
            class="avatar"
          />
        {:else}
          <span class="avatar initials">{initials}</span>
        {/if}

        <span class="user-name">{displayName}</span>
      </a>

      <button class="logout" onclick={logout}>
        Log out
      </button>
    {:else}
      <a href="/login" class="login">Log in</a>
      <a href="/signup" class="signup">Sign up</a>
    {/if}
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
    gap: 20px;
  }

  .logo {
    font-size: 25px;
    font-weight: 800;
    color: #26364a;
    text-decoration: none;
    white-space: nowrap;
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
    gap: 15px;
  }

  .signup {
    background: #5288d9;
    color: white;
    padding: 10px 18px;
    border-radius: 8px;
    text-decoration: none;
    font-size: 14px;
  }

  .profile-link {
    display: flex;
    align-items: center;
    gap: 9px;
    text-decoration: none;
    color: #26364a;
    font-size: 14px;
    font-weight: 600;
  }

  .avatar {
    width: 36px;
    height: 36px;
    border-radius: 50%;
    object-fit: cover;
  }

  .initials {
    display: flex;
    align-items: center;
    justify-content: center;
    background: #5288d9;
    color: white;
    font-size: 13px;
  }

  .user-name {
    max-width: 120px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .logout {
    border: 1px solid #e1e7ef;
    background: white;
    color: #526174;
    padding: 9px 13px;
    border-radius: 8px;
    cursor: pointer;
    font-size: 13px;
  }

  .logout:hover {
    background: #f3f6fa;
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

  @media (max-width: 500px) {
    .user-name {
      display: none;
    }

    .navbar {
      gap: 10px;
    }
  }
</style>
