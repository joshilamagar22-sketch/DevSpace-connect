
<script>
  import { onMount } from 'svelte';
  import { supabase } from '../../lib/supabaseClient.js';

  let searchTerm = $state('');
  let selectedLanguage = $state('All');
  let selectedRepository = $state(null);

  let repositories = $state([]);
  let loading = $state(true);
  let error = $state('');

  let notification = $state('');
  let notificationType = $state('success');

  onMount(() => {
    loadRepositories();
  });

  function showNotification(message, type = 'success') {
    notification = message;
    notificationType = type;
  }

  async function loadRepositories() {
    loading = true;
    error = '';

    const {
      data: { user },
      error: authError
    } = await supabase.auth.getUser();

    if (authError) {
      console.error(authError);
    }

    const { data, error: fetchError } = await supabase
      .from('repositories')
      .select(`
        id,
        name,
        description,
        category,
        github_url,
        user_id,
        created_at,
        profiles (
          full_name,
          username,
          avatar_url
        )
      `)
      .order('created_at', { ascending: false });

    if (fetchError) {
      console.error('Error loading repositories:', fetchError);
      error = 'Could not load repositories. Please try again.';
      repositories = [];
      loading = false;
      return;
    }

    const { data: likes, error: likesError } = await supabase
      .from('repository_likes')
      .select('repository_id, user_id');

    if (likesError) {
      console.error('Likes loading error:', likesError);
    }

    repositories = (data || []).map((repo) => {
      const profile = Array.isArray(repo.profiles)
        ? repo.profiles[0]
        : repo.profiles;

      const repoLikes = likesError
        ? []
        : (likes || []).filter(
            (like) => like.repository_id === repo.id
          );

      return {
        ...repo,
        language: repo.category || 'Other',
        owner: profile?.username || 'developer',
        ownerName: profile?.full_name || 'Developer',
        avatarUrl: profile?.avatar_url || '',
        githubUrl: repo.github_url,
        likesCount: repoLikes.length,
        isLiked: repoLikes.some(
          (like) => like.user_id === user?.id
        )
      };
    });

    loading = false;
  }

  // SAVE REPOSITORY
  async function saveRepository(repositoryId) {
    const {
      data: { user },
      error: authError
    } = await supabase.auth.getUser();

    if (authError || !user) {
      window.location.href = '/login?redirect=/explore';
      return;
    }

    const { error } = await supabase
      .from('saved_repositories')
      .insert({
        user_id: user.id,
        repository_id: repositoryId
      });

    if (error) {
      if (error.code === '23505') {
        showNotification(
          'You have already saved this repository!',
          'error'
        );
      } else {
        console.error(error);
        showNotification(
          'Could not save repository. Please try again.',
          'error'
        );
      }

      return;
    }

    showNotification('Repository saved successfully!');
  }

  // LIKE AND UNLIKE REPOSITORY
  async function likeRepository(repositoryId) {
    const {
      data: { user },
      error: authError
    } = await supabase.auth.getUser();

    if (authError || !user) {
      window.location.href = '/login?redirect=/explore';
      return;
    }

    const repo = repositories.find(
      (item) => item.id === repositoryId
    );

    if (!repo) return;

    if (repo.isLiked) {
      const { error } = await supabase
        .from('repository_likes')
        .delete()
        .eq('repository_id', repositoryId)
        .eq('user_id', user.id);

      if (error) {
        console.error(error);
        showNotification('Could not remove your like.', 'error');
        return;
      }

      repositories = repositories.map((item) =>
        item.id === repositoryId
          ? {
              ...item,
              isLiked: false,
              likesCount: Math.max(0, item.likesCount - 1)
            }
          : item
      );

      showNotification('Like removed.');
    } else {
      const { error } = await supabase
        .from('repository_likes')
        .insert({
          user_id: user.id,
          repository_id: repositoryId
        });

      if (error) {
        if (error.code === '23505') {
          showNotification('You already liked this repository!', 'error');
        } else {
          console.error(error);
          showNotification('Could not like this repository.', 'error');
        }

        return;
      }

      repositories = repositories.map((item) =>
        item.id === repositoryId
          ? {
              ...item,
              isLiked: true,
              likesCount: item.likesCount + 1
            }
          : item
      );

      showNotification('You liked this repository!');
    }

    if (selectedRepository?.id === repositoryId) {
      selectedRepository = repositories.find(
        (item) => item.id === repositoryId
      );
    }
  }

  let languages = $derived([
    'All',
    ...new Set(repositories.map((repo) => repo.language))
  ]);

  let filteredRepositories = $derived(
    repositories.filter((repo) => {
      const search = searchTerm.toLowerCase();

      const matchesSearch =
        repo.name.toLowerCase().includes(search) ||
        (repo.description || '').toLowerCase().includes(search) ||
        repo.ownerName.toLowerCase().includes(search) ||
        repo.owner.toLowerCase().includes(search);

      const matchesLanguage =
        selectedLanguage === 'All' ||
        repo.language === selectedLanguage;

      return matchesSearch && matchesLanguage;
    })
  );

  function closePopup() {
    selectedRepository = null;
  }

  function formatDate(date) {
    if (!date) return '';

    return new Date(date).toLocaleDateString('en-US', {
      year: 'numeric',
      month: 'short',
      day: 'numeric'
    });
  }
</script>

<svelte:head>
  <title>Explore Repositories | DevSpace</title>
</svelte:head>

<main class="explore-page">

  <section class="page-heading">
    <span class="eyebrow">DISCOVER SOMETHING NEW</span>
    <h1>Explore Projects</h1>
    <p>Discover interesting repositories and get inspired by developers.</p>
  </section>

  <section class="repository-section">

    <div class="filters">
      <input
        type="search"
        placeholder="Search repositories or developers..."
        bind:value={searchTerm}
      />

      <select bind:value={selectedLanguage}>
        {#each languages as language}
          <option value={language}>
            {language === 'All' ? 'All Categories' : language}
          </option>
        {/each}
      </select>
    </div>

    <div class="results-heading">
      <h2>Discover repositories</h2>
      <span>{filteredRepositories.length} projects found</span>
    </div>

    {#if loading}
      <p class="empty-state">Loading repositories...</p>

    {:else if error}
      <div class="empty-state">
        <p>{error}</p>
        <button class="retry-button" onclick={loadRepositories}>
          Try Again
        </button>
      </div>

    {:else if filteredRepositories.length === 0}
      <p class="empty-state">
        No repositories found. Try another search.
      </p>

    {:else}
      <div class="repo-grid">

        {#each filteredRepositories as repo (repo.id)}
          <article class="repo-card">

            <div class="card-top">
              <span class="repo-icon">⌘</span>
              <span class="category">{repo.language}</span>
            </div>

            <h3>{repo.name}</h3>

            <p class="description">
              {repo.description || 'No description provided.'}
            </p>

            <div class="owner">
              {#if repo.avatarUrl}
                <img src={repo.avatarUrl} alt="Developer avatar" />
              {:else}
                <div class="avatar">
                  {repo.ownerName.charAt(0).toUpperCase()}
                </div>
              {/if}

              <span>
                {repo.ownerName}
                <small>@{repo.owner}</small>
              </span>
            </div>

            <div class="like-count">
              <span>⭐ {repo.likesCount}</span>
              <span>{repo.likesCount === 1 ? 'Like' : 'Likes'}</span>
            </div>

            <div class="card-bottom">
              <span>{formatDate(repo.created_at)}</span>

              <div class="card-actions">

                <button
                  class="like-button"
                  class:liked={repo.isLiked}
                  onclick={() => likeRepository(repo.id)}
                >
                  {repo.isLiked ? '★ Liked' : '☆ Like'}
                </button>

                <button
                  class="save-button"
                  onclick={() => saveRepository(repo.id)}
                >
                  ☆ Save
                </button>

                <button
                  class="details-button"
                  onclick={() => selectedRepository = repo}
                >
                  Details →
                </button>

              </div>
            </div>

          </article>
        {/each}

      </div>
    {/if}

  </section>

  {#if notification}
    <div class="notification" class:error={notificationType === 'error'}>
      <span>{notificationType === 'success' ? '✓' : '!'}</span>
      <p>{notification}</p>
      <button onclick={() => notification = ''}>×</button>
    </div>
  {/if}

</main>

{#if selectedRepository}
  <div class="modal-overlay" role="presentation" onclick={closePopup}>

    <div
      class="modal-card"
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      onclick={(event) => event.stopPropagation()}
    >
      <button class="modal-close" onclick={closePopup}>×</button>

      <span class="eyebrow">REPOSITORY DETAILS</span>

      <h2 id="modal-title">{selectedRepository.name}</h2>

      <p>
        {selectedRepository.description || 'No description provided.'}
      </p>

      <p>Developer: @{selectedRepository.owner}</p>
      <p>Category: {selectedRepository.language}</p>
      <p>Posted: {formatDate(selectedRepository.created_at)}</p>

      <p class="modal-likes">
        ⭐ {selectedRepository.likesCount} Likes
      </p>

      <button
        class="modal-like-button"
        class:liked={selectedRepository.isLiked}
        onclick={() => likeRepository(selectedRepository.id)}
      >
        {selectedRepository.isLiked ? '★ Unlike' : '☆ Like Repository'}
      </button>

      <a
        href={selectedRepository.githubUrl}
        target="_blank"
        rel="noreferrer"
      >
        View on GitHub ↗
      </a>

    </div>
  </div>
{/if}

<style>
  .explore-page {
    min-height: 75vh;
    padding: 65px 6% 90px;
    color: #26364a;
  }

  .page-heading {
    text-align: center;
    margin-bottom: 55px;
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

  .page-heading p {
    color: #778397;
    line-height: 1.7;
  }

  .repository-section {
    max-width: 1150px;
    margin: auto;
  }

  .filters {
    display: flex;
    gap: 15px;
    margin-bottom: 35px;
  }

  .filters input,
  .filters select {
    padding: 13px 15px;
    border: 1px solid #e1e7ef;
    border-radius: 9px;
    background: white;
    color: #26364a;
    font-size: 14px;
  }

  .filters input {
    flex: 1;
  }

  .results-heading {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 15px;
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

  .owner {
    display: flex;
    align-items: center;
    gap: 10px;
    margin: 22px 0;
    font-size: 13px;
    font-weight: 600;
  }

  .owner img,
  .avatar {
    width: 35px;
    height: 35px;
    border-radius: 50%;
    object-fit: cover;
  }

  .avatar {
    display: grid;
    place-items: center;
    background: #edf4ff;
    color: #5288d9;
  }

  .owner small {
    display: block;
    margin-top: 3px;
    color: #8994a5;
    font-weight: 400;
  }

  .like-count {
    display: flex;
    align-items: center;
    gap: 7px;
    margin-bottom: 16px;
    color: #778397;
    font-size: 13px;
  }

  .like-count span:first-child {
    color: #d69a36;
    font-weight: 700;
  }

  .card-bottom {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 12px;
    padding-top: 16px;
    border-top: 1px solid #edf0f4;
  }

  .card-bottom > span {
    color: #8994a5;
    font-size: 12px;
  }

  .card-actions {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    gap: 9px;
  }

  .card-actions button {
    padding: 8px 10px;
    border: 1px solid #dce5f2;
    border-radius: 7px;
    background: white;
    color: #5288d9;
    font-size: 12px;
    font-weight: 600;
    cursor: pointer;
  }

  .card-actions button:hover {
    background: #edf4ff;
  }

  .card-actions .like-button.liked,
  .modal-like-button.liked {
    border-color: #f1d5a1;
    background: #fff7e8;
    color: #bd8525;
  }

  .empty-state {
    text-align: center;
    padding: 45px;
    color: #778397;
  }

  .retry-button {
    margin-top: 12px;
    border: none;
    background: transparent;
    color: #5288d9;
    font-weight: 600;
    cursor: pointer;
  }

  .notification {
    position: fixed;
    top: 90px;
    right: 25px;
    z-index: 2000;
    display: flex;
    align-items: center;
    gap: 12px;
    min-width: 280px;
    max-width: calc(100vw - 40px);
    padding: 16px 18px;
    border: 1px solid #b9e5cc;
    border-radius: 12px;
    background: #f0fff5;
    color: #247448;
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
  }

  .notification.error {
    border-color: #f2c2c2;
    background: #fff2f2;
    color: #c03939;
  }

  .notification p {
    flex: 1;
    margin: 0;
    font-size: 14px;
  }

  .notification > span {
    font-size: 20px;
    font-weight: bold;
  }

  .notification button {
    border: none;
    background: transparent;
    color: inherit;
    font-size: 21px;
    cursor: pointer;
  }

  .modal-overlay {
    position: fixed;
    inset: 0;
    z-index: 1000;
    display: grid;
    place-items: center;
    padding: 20px;
    background: rgba(20, 30, 45, 0.55);
  }

  .modal-card {
    position: relative;
    width: 100%;
    max-width: 480px;
    padding: 35px;
    border-radius: 16px;
    background: white;
    color: #26364a;
  }

  .modal-card h2 {
    margin: 15px 0;
  }

  .modal-card p {
    color: #778397;
    line-height: 1.7;
  }

  .modal-card a {
    display: inline-block;
    margin-top: 15px;
    color: #5288d9;
    text-decoration: none;
    font-weight: 600;
  }

  .modal-likes {
    font-weight: 700;
  }

  .modal-like-button {
    display: block;
    margin-top: 12px;
    padding: 10px 15px;
    border: 1px solid #dce5f2;
    border-radius: 8px;
    background: white;
    color: #5288d9;
    font-weight: 600;
    cursor: pointer;
  }

  .modal-close {
    position: absolute;
    top: 12px;
    right: 18px;
    border: none;
    background: transparent;
    font-size: 26px;
    cursor: pointer;
  }

  @media (max-width: 650px) {
    .repo-grid {
      grid-template-columns: 1fr;
    }

    .filters {
      flex-direction: column;
    }

    .page-heading h1 {
      font-size: 34px;
    }
  }
</style>
