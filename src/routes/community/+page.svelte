
<script>
  import { onMount } from 'svelte';
  import { supabase } from '../../lib/supabaseClient.js';

  let developers = $state([]);
  let loading = $state(true);
  let error = $state('');

  async function loadDevelopers() {
    loading = true;
    error = '';

    const { data, error: fetchError } = await supabase
      .from('profiles')
      .select('id, full_name, username, avatar_url, bio, skills, github_username')
      .not('github_username', 'is', null)
      .order('created_at', { ascending: false });

    if (fetchError) {
      error = fetchError.message;
    } else {
      developers = data || [];
    }

    loading = false;
  }

  onMount(() => {
    loadDevelopers();
  });
</script>

<svelte:head>
  <title>Community | DevSpace</title>
  <meta
    name="description"
    content="Discover developers and explore their GitHub profiles on DevSpace."
  />
</svelte:head>

<main class="community-page">
  <section class="community-heading">
    <p class="eyebrow">DEVSPACE COMMUNITY</p>
    <h1>Meet the Developers</h1>
    <p>
      Discover developers in our community and explore what they're building.
    </p>
  </section>

  {#if loading}
    <div class="message">
      <p>Loading developers...</p>
    </div>

  {:else if error}
    <div class="message">
      <p>Could not load developers: {error}</p>
      <button onclick={loadDevelopers}>Try Again</button>
    </div>

  {:else if developers.length === 0}
    <div class="empty-state">
      <h2>No developers yet</h2>
      <p>
        Connect your GitHub account to become part of the DevSpace community.
      </p>
      <a href="/profile">Go to My Profile</a>
    </div>

  {:else}
    <div class="developer-count">
      {developers.length} developer{developers.length === 1 ? '' : 's'} connected
    </div>

    <section class="developer-grid">
      {#each developers as developer (developer.id)}
        <article class="developer-card">
          <div class="developer-top">
            {#if developer.avatar_url}
              <img
                class="developer-avatar"
                src={developer.avatar_url}
                alt={developer.full_name}
              />
            {:else}
              <div class="developer-avatar fallback-avatar">
                {developer.full_name?.charAt(0)?.toUpperCase() || 'D'}
              </div>
            {/if}

            <div class="developer-info">
              <h2>{developer.full_name || 'Developer'}</h2>
              <p>@{developer.username || 'devspace-user'}</p>
            </div>
          </div>

          {#if developer.bio}
            <p class="developer-bio">{developer.bio}</p>
          {:else}
            <p class="developer-bio muted">No bio added yet.</p>
          {/if}

          {#if developer.skills?.length}
            <div class="skills">
              {#each developer.skills.slice(0, 4) as skill}
                <span>{skill}</span>
              {/each}
            </div>
          {/if}

          <div class="github-info">
            <span class="github-icon">⌘</span>
            <span>@{developer.github_username}</span>
          </div>

          <div class="developer-actions">
            <a
              class="github-button"
              href={`https://github.com/${encodeURIComponent(developer.github_username)}`}
              target="_blank"
              rel="noopener noreferrer"
            >
              View GitHub
            </a>

            <a
              class="profile-button"
              href={`/profile?id=${developer.id}`}
            >
              DevSpace Profile
            </a>
          </div>
        </article>
      {/each}
    </section>
  {/if}
</main>

<style>
  .community-page {
    max-width: 1200px;
    margin: 0 auto;
    padding: 65px 24px 90px;
    min-height: 75vh;
  }

  .community-heading {
    text-align: center;
    margin-bottom: 45px;
  }

  .eyebrow {
    color: #8b5cf6;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 2px;
    margin-bottom: 12px;
  }

  .community-heading h1 {
    font-size: clamp(30px, 5vw, 44px);
    margin-bottom: 12px;
  }

  .community-heading > p:last-child {
    color: #888;
    font-size: 15px;
  }

  .developer-count {
    color: #888;
    font-size: 13px;
    margin-bottom: 18px;
  }

  .developer-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(270px, 1fr));
    gap: 22px;
  }

  .developer-card {
    border: 1px solid #333;
    border-radius: 16px;
    padding: 22px;
    background: #171717;
    transition: transform 0.2s, border-color 0.2s;
  }

  .developer-card:hover {
    transform: translateY(-4px);
    border-color: #8b5cf6;
  }

  .developer-top {
    display: flex;
    align-items: center;
    gap: 14px;
    margin-bottom: 18px;
  }

  .developer-avatar {
    width: 54px;
    height: 54px;
    border-radius: 50%;
    object-fit: cover;
  }

  .fallback-avatar {
    display: flex;
    align-items: center;
    justify-content: center;
    background: #292044;
    color: #c4b5fd;
    font-size: 22px;
    font-weight: 700;
  }

  .developer-info h2 {
    font-size: 17px;
    margin: 0 0 5px;
  }

  .developer-info p {
    color: #999;
    font-size: 13px;
    margin: 0;
  }

  .developer-bio {
    color: #ccc;
    font-size: 14px;
    line-height: 1.6;
    min-height: 42px;
    margin-bottom: 15px;
  }

  .muted {
    color: #777;
  }

  .skills {
    display: flex;
    flex-wrap: wrap;
    gap: 7px;
    margin-bottom: 17px;
  }

  .skills span {
    background: #292044;
    color: #c4b5fd;
    padding: 5px 10px;
    border-radius: 20px;
    font-size: 11px;
  }

  .github-info {
    display: flex;
    align-items: center;
    gap: 9px;
    color: #ccc;
    font-size: 13px;
    padding: 13px 0;
    border-top: 1px solid #333;
  }

  .github-icon {
    font-size: 20px;
  }

  .developer-actions {
    display: flex;
    gap: 10px;
    margin-top: 10px;
  }

  .developer-actions a {
    flex: 1;
    text-align: center;
    text-decoration: none;
    padding: 10px 8px;
    border-radius: 8px;
    font-size: 12px;
    font-weight: 600;
  }

  .github-button {
    background: #8b5cf6;
    color: white;
  }

  .profile-button {
    border: 1px solid #555;
    color: white;
  }

  .message,
  .empty-state {
    text-align: center;
    padding: 55px 20px;
    color: #aaa;
  }

  .empty-state h2 {
    color: white;
    margin-bottom: 10px;
  }

  .empty-state a,
  .message button {
    display: inline-block;
    margin-top: 18px;
    padding: 10px 18px;
    background: #8b5cf6;
    color: white;
    border: none;
    border-radius: 8px;
    text-decoration: none;
    cursor: pointer;
  }

  @media (max-width: 600px) {
    .community-page {
      padding: 45px 16px 65px;
    }

    .developer-actions {
      flex-direction: column;
    }
  }
</style>
