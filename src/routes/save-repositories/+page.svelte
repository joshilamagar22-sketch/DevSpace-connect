
<script>
  import { onMount } from 'svelte';
  import { supabase } from '../../lib/supabaseClient.js';

  let repositories = $state([]);
  let loading = $state(true);
  let error = $state('');

  onMount(() => {
    loadSavedRepositories();
  });

  async function loadSavedRepositories() {
    loading = true;
    error = '';

    const { data: { user }, error: authError } =
      await supabase.auth.getUser();

    if (authError || !user) {
      window.location.href = '/login?redirect=/saved-repositories';
      return;
    }

    const { data, error: fetchError } = await supabase
      .from('saved_repositories')
      .select(`
        id,
        created_at,
        repositories (
          id,
          name,
          description,
          category,
          github_url,
          user_id
        )
      `)
      .eq('user_id', user.id)
      .order('created_at', { ascending: false });

    if (fetchError) {
      console.error(fetchError);
      error = 'Could not load your saved repositories.';
      repositories = [];
    } else {
      repositories = (data || [])
        .filter((item) => item.repositories)
        .map((item) => ({
          ...item.repositories,
          savedId: item.id,
          savedAt: item.created_at
        }));
    }

    loading = false;
  }

  async function removeSavedRepository(savedId) {
    const { error: deleteError } = await supabase
      .from('saved_repositories')
      .delete()
      .eq('id', savedId);

    if (deleteError) {
      console.error(deleteError);
      error = 'Could not remove this repository.';
      return;
    }

    repositories = repositories.filter(
      (repo) => repo.savedId !== savedId
    );
  }
</script>

<svelte:head>
  <title>My Saved Repositories | DevSpace</title>
</svelte:head>

<main class="saved-page">
  <section class="heading">
    <span class="eyebrow">YOUR COLLECTION</span>
    <h1>Saved Repositories</h1>
    <p>All your bookmarked developer projects in one place.</p>
  </section>

  <section class="repository-section">
    <div class="results-heading">
      <h2>My Bookmarks</h2>
      <span>{repositories.length} saved projects</span>
    </div>

    {#if loading}
      <p class="empty-state">Loading your saved repositories...</p>

    {:else if error}
      <p class="empty-state">{error}</p>
      <button class="retry-button" onclick={loadSavedRepositories}>
        Try Again
      </button>

    {:else if repositories.length === 0}
      <div class="empty-state">
        <div class="empty-icon">☆</div>
        <h3>No saved repositories yet</h3>
        <p>Explore projects and save the ones you like!</p>
        <a href="/explore">Explore Projects →</a>
      </div>

    {:else}
      <div class="repo-grid">
        {#each repositories as repo (repo.savedId)}
          <article class="repo-card">
            <div class="card-top">
              <span class="repo-icon">⌘</span>
              <span class="category">{repo.category || 'Other'}</span>
            </div>

            <h3>{repo.name}</h3>

            <p class="description">
              {repo.description || 'No description provided.'}
            </p>

            <div class="card-bottom">
              <a
                href={repo.github_url}
                target="_blank"
                rel="noreferrer"
              >
                View on GitHub ↗
              </a>

              <button
                class="remove-button"
                onclick={() => removeSavedRepository(repo.savedId)}
              >
                Remove
              </button>
            </div>
          </article>
        {/each}
      </div>
    {/if}
  </section>
</main>

<style>
  .saved-page {
    min-height: 75vh;
    padding: 65px 6% 90px;
    color: #26364a;
  }

  .heading {
    text-align: center;
    margin-bottom: 55px;
  }

  .eyebrow {
    color: #5288d9;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 2px;
  }

  .heading h1 {
    margin: 15px 0 10px;
    font-size: 42px;
  }

  .heading p {
    color: #778397;
    line-height: 1.7;
  }

  .repository-section {
    max-width: 1150px;
    margin: auto;
  }

  .results-heading {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 22px;
  }

  .results-heading h2 {
    font-size: 21px;
  }

  .results-heading span {
    color: #8994a5;
    font-size: 13px;
  }

  .repo-grid {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 22px;
  }

  .repo-card {
    padding: 25px;
    border: 1px solid #e7ebf2;
    border-radius: 14px;
    background: white;
    transition: 0.2s ease;
  }

  .repo-card:hover {
    transform: translateY(-4px);
    box-shadow: 0 8px 25px rgba(30, 50, 80, 0.08);
  }

  .card-top {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .repo-icon {
    font-size: 25px;
    color: #5288d9;
  }

  .category {
    padding: 6px 11px;
    border-radius: 20px;
    background: #edf4ff;
    color: #5288d9;
    font-size: 12px;
  }

  .repo-card h3 {
    margin: 18px 0 10px;
    font-size: 20px;
  }

  .description {
    min-height: 48px;
    color: #778397;
    font-size: 14px;
    line-height: 1.7;
  }

  .card-bottom {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 10px;
    margin-top: 22px;
    padding-top: 16px;
    border-top: 1px solid #edf0f4;
  }

  .card-bottom a {
    color: #5288d9;
    text-decoration: none;
    font-size: 13px;
    font-weight: 600;
  }

  .remove-button,
  .retry-button {
    border: none;
    background: transparent;
    color: #d65b5b;
    font-weight: 600;
    cursor: pointer;
  }

  .empty-state {
    padding: 45px;
    text-align: center;
    color: #778397;
  }

  .empty-icon {
    font-size: 45px;
    color: #5288d9;
  }

  .empty-state h3 {
    color: #26364a;
  }

  .empty-state a {
    display: inline-block;
    margin-top: 12px;
    color: #5288d9;
    text-decoration: none;
    font-weight: 600;
  }

  @media (max-width: 650px) {
    .repo-grid {
      grid-template-columns: 1fr;
    }

    .heading h1 {
      font-size: 34px;
    }
  }
</style>
