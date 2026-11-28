
<script>
  import { onMount } from 'svelte';
  import { supabase } from '../../lib/supabaseClient.js';

  let repositories = $state([]);
  let loading = $state(true);
  let error = $state('');
  let connected = $state(false);

  onMount(async () => {
    await fetchRepositories();
  });

  async function fetchRepositories() {
    loading = true;
    error = '';

    try {
      // Check logged-in user
      const {
        data: { user },
        error: userError
      } = await supabase.auth.getUser();

      if (userError || !user) {
        window.location.href = '/login?redirect=/my-repositories';
        return;
      }

      // Fetch this user's repositories
      const { data, error: fetchError } = await supabase
        .from('repositories')
        .select('*')
        .eq('user_id', user.id)
        .order('created_at', { ascending: false });

      if (fetchError) {
        error = fetchError.message;
        return;
      }

      repositories = data || [];

    } catch (err) {
      error = 'Something went wrong while loading your repositories.';
    } finally {
      loading = false;
    }
  }

  function connectGitHub() {
    // Temporary demo interaction.
    // Real GitHub OAuth will be implemented later.
    connected = true;
  }
</script>

<svelte:head>
  <title>My Repositories | DevSpace</title>
</svelte:head>

<main class="my-page">

  <section class="page-heading">
    <span class="eyebrow">YOUR DEVELOPER SPACE</span>
    <h1>My Repositories</h1>
    <p>Manage and showcase your GitHub projects in one place.</p>
  </section>

  <section class="dashboard">

    <div class="dashboard-heading">
      <div>
        <h2>Your projects</h2>
        <p>Your personal repository collection.</p>
      </div>

      <a href="/add-repository" class="add-button">
        + Add Repository
      </a>
    </div>

    {#if loading}
      <div class="status-message">
        Loading your repositories...
      </div>

    {:else if error}
      <div class="status-message error-message">
        {error}
        <button class="retry-button" onclick={fetchRepositories}>
          Try again
        </button>
      </div>

    {:else if repositories.length === 0}
      <div class="empty-state">
        <div class="empty-icon">⌘</div>
        <h2>No repositories yet</h2>
        <p>
          Your developer journey starts here.
          Add your first GitHub repository to showcase your work!
        </p>
        <a href="/add-repository" class="add-button">
          + Add Your First Repository
        </a>
      </div>

    {:else}
      <div class="repository-grid">

        {#each repositories as repo (repo.id)}
          <article class="repository-card">

            <div class="card-top">
              <span class="repo-icon">⌘</span>
              <span class="category">{repo.category || 'Other'}</span>
            </div>

            <h3>{repo.name}</h3>

            <p class="description">
              {repo.description || 'No description provided.'}
            </p>

            <div class="card-bottom">
              <span class="date">
                Added {new Date(repo.created_at).toLocaleDateString()}
              </span>

              <a
                href={repo.github_url}
                target="_blank"
                rel="noopener noreferrer"
              >
                View on GitHub ↗
              </a>
            </div>

          </article>
        {/each}

      </div>
    {/if}

    <div class="connect-card">
      <div class="github-icon">⌘</div>

      {#if connected}
        <h2>GitHub connected!</h2>
        <p>
          This is currently a demo connection.
          Real GitHub authentication is coming next.
        </p>
      {:else}
        <h2>Connect your GitHub account</h2>
        <p>
          Connect GitHub to display your repositories
          and showcase your work on DevSpace.
        </p>
      {/if}

      <button class="github-button" onclick={connectGitHub}>
        {connected ? 'Connected ✓' : 'Connect GitHub'}
      </button>
    </div>

  </section>
</main>

<style>
  .my-page {
    min-height: 75vh;
    padding: 65px 6% 90px;
    color: #26364a;
  }

  .page-heading {
    text-align: center;
    margin-bottom: 60px;
  }

  .eyebrow {
    color: #5288d9;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 2px;
  }

  .page-heading h1 {
    font-size: 42px;
    margin: 15px 0 10px;
  }

  .page-heading p,
  .dashboard-heading p {
    color: #778397;
    line-height: 1.7;
  }

  .dashboard {
    max-width: 1000px;
    margin: auto;
  }

  .dashboard-heading {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 20px;
    margin-bottom: 25px;
  }

  .dashboard-heading h2 {
    margin-bottom: 5px;
  }

  .dashboard-heading p {
    margin: 0;
    font-size: 14px;
  }

  .add-button,
  .github-button {
    border: none;
    border-radius: 8px;
    padding: 12px 18px;
    background: #5288d9;
    color: white;
    font-weight: 600;
    cursor: pointer;
    text-decoration: none;
    display: inline-block;
  }

  .add-button:hover,
  .github-button:hover {
    background: #3d70bd;
  }

  .repository-grid {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 22px;
    margin-bottom: 45px;
  }

  .repository-card {
    border: 1px solid #e8edf4;
    border-radius: 13px;
    padding: 25px;
    background: white;
    transition: transform 0.2s, box-shadow 0.2s;
  }

  .repository-card:hover {
    transform: translateY(-4px);
    box-shadow: 0 8px 25px rgba(38, 54, 74, 0.08);
  }

  .card-top {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 18px;
  }

  .repo-icon {
    font-size: 25px;
  }

  .category {
    background: #edf4ff;
    color: #5288d9;
    padding: 6px 10px;
    border-radius: 20px;
    font-size: 11px;
    font-weight: 600;
  }

  .repository-card h3 {
    font-size: 20px;
    margin: 0 0 12px;
    overflow-wrap: anywhere;
  }

  .description {
    color: #778397;
    font-size: 14px;
    line-height: 1.7;
    min-height: 48px;
    overflow-wrap: anywhere;
  }

  .card-bottom {
    border-top: 1px solid #edf0f5;
    padding-top: 15px;
    margin-top: 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 10px;
    flex-wrap: wrap;
  }

  .date {
    color: #929cac;
    font-size: 12px;
  }

  .card-bottom a {
    color: #5288d9;
    font-size: 13px;
    font-weight: 600;
    text-decoration: none;
  }

  .card-bottom a:hover {
    text-decoration: underline;
  }

  .status-message,
  .empty-state {
    text-align: center;
    padding: 55px 25px;
    border: 1px solid #e8edf4;
    border-radius: 15px;
    background: #f9fbfe;
    color: #778397;
    margin-bottom: 40px;
  }

  .error-message {
    color: #c03939;
  }

  .retry-button {
    display: block;
    margin: 15px auto 0;
    padding: 10px 18px;
    border: none;
    border-radius: 8px;
    background: #5288d9;
    color: white;
    cursor: pointer;
  }

  .empty-icon {
    font-size: 42px;
    margin-bottom: 12px;
  }

  .empty-state h2 {
    color: #26364a;
  }

  .empty-state p {
    color: #778397;
    max-width: 430px;
    margin: 12px auto 25px;
    line-height: 1.7;
    font-size: 14px;
  }

  .connect-card {
    border: 1px solid #e8edf4;
    border-radius: 15px;
    padding: 45px 25px;
    text-align: center;
    background: #f9fbfe;
  }

  .github-icon {
    font-size: 42px;
    margin-bottom: 15px;
  }

  .connect-card p {
    color: #778397;
    max-width: 450px;
    margin: 12px auto 25px;
    line-height: 1.7;
    font-size: 14px;
  }

  @media (max-width: 650px) {
    .repository-grid {
      grid-template-columns: 1fr;
    }

    .dashboard-heading {
      align-items: flex-start;
      flex-direction: column;
    }

    .page-heading h1 {
      font-size: 34px;
    }
  }
</style>
