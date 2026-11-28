
<script>
  import { onMount } from 'svelte';
  import { supabase } from '../../lib/supabaseClient.js';

  let loading = $state(true);
  let isAdmin = $state(false);
  let error = $state('');
  let deletingId = $state(null);
  let activeSection = $state('overview');

  let profiles = $state([]);
  let repositories = $state([]);
  let userSearch = $state('');
  let adminIds = $state([]);

  let filteredUsers = $derived(
    profiles.filter((user) => {
      const search = userSearch.toLowerCase();

      return (
        (user.full_name || '').toLowerCase().includes(search) ||
        (user.username || '').toLowerCase().includes(search) ||
        (user.github_username || '').toLowerCase().includes(search)
      );
    })
  );

  let stats = $state({
    users: 0,
    repositories: 0,
    githubUsers: 0,
    likes: 0
  });

  const navigation = [
    { id: 'overview', icon: '◫', label: 'Overview' },
    { id: 'users', icon: '♙', label: 'Users' },
    { id: 'repositories', icon: '⌘', label: 'Repositories' },
    { id: 'community', icon: '◎', label: 'Community' }
  ];

  async function loadDashboard() {
    loading = true;
    error = '';

    const {
      data: { user },
      error: authError
    } = await supabase.auth.getUser();

    if (authError || !user) {
      window.location.href = '/login?redirect=/admin';
      return;
    }

    const { data: adminResult, error: adminError } =
      await supabase.rpc('is_admin');

    if (adminError || adminResult !== true) {
      isAdmin = false;
      loading = false;
      return;
    }

    isAdmin = true;

    const [
      profileResult,
      repositoryResult,
      userCount,
      repositoryCount,
      githubCount,
      likesCount
    ] = await Promise.all([
      supabase
        .from('profiles')
        .select(
          'id, full_name, username, avatar_url, bio, skills, github_username, created_at'
        )
        .order('created_at', { ascending: false }),

      supabase
        .from('repositories')
        .select(`
          id, name, description, category, github_url,
          created_at, user_id,
          profiles(full_name, username)
        `)
        .order('created_at', { ascending: false }),

      supabase
        .from('profiles')
        .select('id', { count: 'exact', head: true }),

      supabase
        .from('repositories')
        .select('id', { count: 'exact', head: true }),

      supabase
        .from('profiles')
        .select('id', { count: 'exact', head: true })
        .not('github_username', 'is', null),

      supabase
        .from('repository_likes')
        .select('id', { count: 'exact', head: true })
    ]);

    if (
      profileResult.error ||
      repositoryResult.error ||
      userCount.error ||
      repositoryCount.error ||
      githubCount.error ||
      likesCount.error
    ) {
      error = 'Could not load dashboard data. Please refresh.';
    } else {
      profiles = profileResult.data || [];
      repositories = repositoryResult.data || [];

      // Check which users are administrators
      const adminResults = await Promise.all(
        profiles.map(async (person) => {
          const { data, error: checkError } = await supabase.rpc(
            'is_user_admin',
            {
              check_user_id: person.id
            }
          );

          if (checkError) {
            console.error(
              'Admin check failed:',
              person.id,
              checkError
            );
            return null;
          }

          return data === true ? person.id : null;
        })
      );

      adminIds = adminResults.filter(Boolean);

      console.log('Admin IDs:', adminIds);
      console.log(
        'All profile IDs:',
        profiles.map((person) => person.id)
      );

      stats = {
        users: userCount.count || 0,
        repositories: repositoryCount.count || 0,
        githubUsers: githubCount.count || 0,
        likes: likesCount.count || 0
      };
    }

    loading = false;
  }

  async function deleteRepository(repository) {
    const confirmed = window.confirm(
      `Delete "${repository.name}" permanently?`
    );

    if (!confirmed) return;

    deletingId = repository.id;
    error = '';

    const { error: deleteError } = await supabase
      .from('repositories')
      .delete()
      .eq('id', repository.id);

    if (deleteError) {
      error = deleteError.message;
    } else {
      repositories = repositories.filter(
        (item) => item.id !== repository.id
      );

      stats.repositories = Math.max(
        0,
        stats.repositories - 1
      );
    }

    deletingId = null;
  }

  onMount(() => {
    loadDashboard();
  });
</script>

<svelte:head>
  <title>Admin Dashboard | DevSpace</title>
</svelte:head>

{#if loading}
  <main class="loading-screen">
    <div class="loader"></div>
    <p>Loading DevSpace Admin...</p>
  </main>

{:else if !isAdmin}
  <main class="access-denied">
    <div class="lock">🔒</div>
    <h1>Access Denied</h1>
    <p>You don't have permission to access this dashboard.</p>
    <a href="/">Return to DevSpace</a>
  </main>

{:else}
  <div class="admin-layout">

    <aside class="sidebar">
      <a href="/" class="brand">
        <span class="brand-icon">⌘</span>
        <span>Dev<span>Space</span></span>
      </a>

      <div class="sidebar-label">ADMIN PANEL</div>

      <nav>
        {#each navigation as item}
          <button
            class:active={activeSection === item.id}
            onclick={() => activeSection = item.id}
          >
            <span class="nav-icon">{item.icon}</span>
            {item.label}

            {#if activeSection === item.id}
              <span class="active-dot"></span>
            {/if}
          </button>
        {/each}
      </nav>

      <div class="sidebar-bottom">
        <div class="admin-badge">
          <span class="admin-avatar">A</span>
          <div>
            <strong>Administrator</strong>
            <small>DevSpace Admin</small>
          </div>
        </div>

        <a href="/" class="back-link">← Back to DevSpace</a>
      </div>
    </aside>

    <main class="admin-main">
      <header class="topbar">
        <div>
          <p class="topbar-label">DEVSPACE / ADMIN</p>
          <h1>
            {navigation.find(item => item.id === activeSection)?.label}
          </h1>
        </div>

        <button class="refresh-button" onclick={loadDashboard}>
          ↻ Refresh Data
        </button>
      </header>

      {#if error}
        <div class="error-message">{error}</div>
      {/if}

      {#if activeSection === 'overview'}

        <section class="welcome-panel">
          <div>
            <span class="welcome-label">CONTROL CENTER</span>
            <h2>Welcome to DevSpace Admin.</h2>
            <p>Monitor your developer platform and manage its content.</p>
          </div>

          <div class="welcome-symbol">⌘</div>
        </section>

        <div class="section-title">
          <h2>Platform Overview</h2>
          <p>Current platform statistics</p>
        </div>

        <section class="stats-grid">
          <article class="stat-card">
            <span>Total Users</span>
            <h2>{stats.users}</h2>
            <p>Registered developers</p>
          </article>

          <article class="stat-card">
            <span>Repositories</span>
            <h2>{stats.repositories}</h2>
            <p>Published projects</p>
          </article>

          <article class="stat-card">
            <span>GitHub Connected</span>
            <h2>{stats.githubUsers}</h2>
            <p>Connected developers</p>
          </article>

          <article class="stat-card">
            <span>Total Likes</span>
            <h2>{stats.likes}</h2>
            <p>Repository likes</p>
          </article>
        </section>

        <section class="quick-section">
          <div class="section-title">
            <h2>Quick Management</h2>
            <p>Jump to a management section</p>
          </div>

          <div class="quick-grid">
            <button onclick={() => activeSection = 'users'}>
              <span>♙</span>
              <strong>Manage Users</strong>
              <small>View registered developers →</small>
            </button>

            <button onclick={() => activeSection = 'repositories'}>
              <span>⌘</span>
              <strong>Manage Repositories</strong>
              <small>Review published projects →</small>
            </button>

            <button onclick={() => activeSection = 'community'}>
              <span>◎</span>
              <strong>Community</strong>
              <small>View GitHub-connected users →</small>
            </button>
          </div>
        </section>

      {:else if activeSection === 'users'}

        <input
          type="text"
          placeholder="Search by name, username or GitHub..."
          bind:value={userSearch}
          class="user-search"
        />

        <section class="content-section">
          <div class="section-title">
            <h2>Registered Users</h2>
            <p>All developers registered on DevSpace.</p>
          </div>

          <div class="table-wrapper">
            <table>
              <thead>
                <tr>
                  <th>Developer</th>
                  <th>Username</th>
                  <th>GitHub</th>
                  <th>Joined</th>
                  <th>Action</th>
                </tr>
              </thead>

              <tbody>
                {#each filteredUsers as person (person.id)}
                  <tr>
                    <td>
                      <div class="user-cell">
                        {#if person.avatar_url}
                          <img src={person.avatar_url} alt="" />
                        {:else}
                          <div class="user-initial">
                            {person.full_name?.charAt(0)?.toUpperCase() || 'D'}
                          </div>
                        {/if}

                        <div class="user-info">
                          <div>{person.full_name || 'Developer'}</div>

                          {#if adminIds.includes(person.id)}
                            <span class="admin-user-tag">ADMIN</span>
                          {/if}
                        </div>
                      </div>
                    </td>

                    <td>@{person.username || '—'}</td>

                    <td>
                      {#if person.github_username}
                        <a
                          href={`https://github.com/${encodeURIComponent(person.github_username)}`}
                          target="_blank"
                          rel="noopener noreferrer"
                        >
                          @{person.github_username}
                        </a>
                      {:else}
                        <span class="muted">Not connected</span>
                      {/if}
                    </td>

                    <td>
                      {new Date(person.created_at).toLocaleDateString()}
                    </td>

                    <td>
                      <a
                        href={`/profile?id=${person.id}`}
                        class="view-profile"
                      >
                        View Profile →
                      </a>
                    </td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>

          {#if filteredUsers.length === 0}
            <p class="empty">
              {profiles.length === 0
                ? 'No users registered yet.'
                : 'No users match your search.'}
            </p>
          {/if}
        </section>

      {:else if activeSection === 'repositories'}

        <section class="content-section">
          <div class="section-title">
            <h2>Repository Management</h2>
            <p>Review and manage all published projects.</p>
          </div>

          {#if repositories.length === 0}
            <p class="empty">No repositories published yet.</p>
          {:else}
            <div class="repository-list">
              {#each repositories as repo (repo.id)}
                <article class="repository-item">
                  <div class="repo-details">
                    <h3>{repo.name}</h3>
                    <p>{repo.description || 'No description provided.'}</p>

                    <div class="repo-meta">
                      <span>{repo.category || 'Other'}</span>
                      <span>By {repo.profiles?.full_name || 'Unknown'}</span>
                      <span>
                        {new Date(repo.created_at).toLocaleDateString()}
                      </span>
                    </div>
                  </div>

                  <div class="repo-actions">
                    <a
                      href={repo.github_url}
                      target="_blank"
                      rel="noopener noreferrer"
                    >
                      View GitHub ↗
                    </a>

                    <button
                      class="delete-button"
                      disabled={deletingId === repo.id}
                      onclick={() => deleteRepository(repo)}
                    >
                      {deletingId === repo.id ? 'Deleting...' : 'Delete'}
                    </button>
                  </div>
                </article>
              {/each}
            </div>
          {/if}
        </section>

      {:else if activeSection === 'community'}

        <section class="content-section">
          <div class="section-title">
            <h2>Developer Community</h2>
            <p>Users who have connected their GitHub accounts.</p>
          </div>

          <div class="community-grid">
            {#each profiles.filter(person => person.github_username) as person (person.id)}
              <article class="community-card">
                {#if person.avatar_url}
                  <img src={person.avatar_url} alt="" />
                {:else}
                  <div class="community-avatar">
                    {person.full_name?.charAt(0)?.toUpperCase() || 'D'}
                  </div>
                {/if}

                <h3>{person.full_name || 'Developer'}</h3>

                <p>@{person.username || 'devspace-user'}</p>

                <a
                  href={`https://github.com/${encodeURIComponent(person.github_username)}`}
                  target="_blank"
                  rel="noopener noreferrer"
                >
                  @{person.github_username} ↗
                </a>
              </article>
            {/each}
          </div>

          {#if !profiles.some(person => person.github_username)}
            <p class="empty">
              No GitHub-connected developers yet.
            </p>
          {/if}
        </section>

      {/if}

      <footer class="admin-footer">
        <span>DevSpace Admin Control Center</span>
        <span>Manage your developer community.</span>
      </footer>
    </main>
  </div>
{/if}

<style>
  :global(body) {
    background: #0b0b10;
  }

  .admin-layout {
    display: flex;
    min-height: 100vh;
    background: #0b0b10;
    color: #f4f2fa;
    font-family: Inter, system-ui, sans-serif;
  }

  /* SIDEBAR */

  .sidebar {
    width: 245px;
    flex-shrink: 0;
    min-height: 100vh;
    padding: 27px 17px;
    background: #111019;
    border-right: 1px solid #292432;
    display: flex;
    flex-direction: column;
  }

  .brand {
    display: flex;
    align-items: center;
    gap: 10px;
    color: white;
    text-decoration: none;
    font-size: 21px;
    font-weight: 800;
    padding: 0 10px;
    margin-bottom: 52px;
  }

  .brand > span:last-child > span {
    color: #a78bfa;
  }

  .brand-icon {
    width: 34px;
    height: 34px;
    display: grid;
    place-items: center;
    background: #33204f;
    color: #c4b5fd;
    border-radius: 9px;
    font-size: 23px;
  }

  .sidebar-label {
    color: #777083;
    font-size: 10px;
    font-weight: 800;
    letter-spacing: 1.7px;
    padding: 0 11px;
    margin-bottom: 14px;
  }

  .sidebar nav {
    display: grid;
    gap: 7px;
  }

  .sidebar nav button {
    display: flex;
    align-items: center;
    gap: 12px;
    width: 100%;
    padding: 13px 12px;
    color: #9992a8;
    background: transparent;
    border: 1px solid transparent;
    border-radius: 9px;
    text-align: left;
    font-size: 13px;
    transition: 0.2s;
  }

  .sidebar nav button:hover {
    background: #1d1928;
    color: white;
  }

  .sidebar nav button.active {
    background: #271d3b;
    color: #d1c2ff;
    border-color: #453064;
  }

  .nav-icon {
    width: 20px;
    font-size: 18px;
    text-align: center;
  }

  .active-dot {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: #a78bfa;
    margin-left: auto;
  }

  .sidebar-bottom {
    margin-top: auto;
    padding-top: 30px;
  }

  .admin-badge {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 13px 9px;
    border-top: 1px solid #292432;
  }

  .admin-avatar {
    width: 34px;
    height: 34px;
    display: grid;
    place-items: center;
    border-radius: 50%;
    background: #392652;
    color: #d8c6ff;
    font-weight: 700;
  }

  .admin-badge strong,
  .admin-badge small {
    display: block;
  }

  .admin-badge strong {
    font-size: 12px;
  }

  .admin-badge small {
    color: #81798f;
    font-size: 10px;
    margin-top: 4px;
  }

  .back-link {
    display: block;
    color: #a99ebd;
    text-decoration: none;
    font-size: 12px;
    padding: 12px 10px;
  }

  .back-link:hover {
    color: white;
  }

  /* MAIN */

  .admin-main {
    flex: 1;
    min-width: 0;
    padding: 35px 38px 25px;
  }

  .topbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 15px;
    margin-bottom: 32px;
  }

  .topbar-label {
    color: #8e80a8;
    font-size: 10px;
    font-weight: 700;
    letter-spacing: 1.7px;
    margin: 0 0 8px;
  }

  .topbar h1 {
    font-size: 27px;
    margin: 0;
    letter-spacing: -0.7px;
  }

  .refresh-button {
    background: #8b5cf6;
    color: white;
    border: 1px solid #a78bfa;
    border-radius: 8px;
    padding: 10px 15px;
    font-size: 12px;
    font-weight: 650;
    cursor: pointer;
  }

  .refresh-button:hover {
    background: #7542df;
  }

  /* OVERVIEW */

  .welcome-panel {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 20px;
    padding: 30px;
    margin-bottom: 35px;
    border: 1px solid #3b2b52;
    border-radius: 14px;
    background: linear-gradient(110deg, #211632, #15131d);
  }

  .welcome-label {
    color: #b49af9;
    font-size: 10px;
    letter-spacing: 2px;
    font-weight: 800;
  }

  .welcome-panel h2 {
    font-size: 24px;
    margin: 12px 0 8px;
  }

  .welcome-panel p {
    color: #a69cb5;
    font-size: 13px;
    margin: 0;
  }

  .welcome-symbol {
    color: #a78bfa;
    font-size: 70px;
    opacity: 0.7;
  }

  .section-title {
    margin-bottom: 19px;
  }

  .section-title h2 {
    font-size: 19px;
    margin: 0 0 6px;
  }

  .section-title p {
    color: #888195;
    font-size: 12px;
    margin: 0;
  }

  .stats-grid {
    display: grid;
    grid-template-columns: repeat(4, minmax(0, 1fr));
    gap: 15px;
  }

  .stat-card {
    padding: 22px;
    border: 1px solid #2b2735;
    border-radius: 12px;
    background: #14131b;
    transition: 0.2s;
  }

  .stat-card:hover {
    border-color: #7952bb;
    transform: translateY(-3px);
  }

  .stat-card span {
    color: #aaa2b8;
    font-size: 11px;
    text-transform: uppercase;
    letter-spacing: 0.7px;
  }

  .stat-card h2 {
    color: #c4b5fd;
    font-size: 34px;
    margin: 14px 0 5px;
  }

  .stat-card p {
    color: #81798e;
    font-size: 11px;
    margin: 0;
  }

  .quick-section {
    margin-top: 40px;
  }

  .quick-grid {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 14px;
  }

  .quick-grid button {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    gap: 10px;
    padding: 20px;
    background: #14131b;
    border: 1px solid #2b2735;
    border-radius: 12px;
    color: white;
    text-align: left;
    cursor: pointer;
  }

  .quick-grid button:hover {
    border-color: #7952bb;
    background: #1b1725;
  }

  .quick-grid button > span {
    color: #b9a2ff;
    font-size: 25px;
  }

  .quick-grid strong {
    font-size: 13px;
  }

  .quick-grid small {
    color: #898194;
    font-size: 11px;
  }

  /* USERS AND TABLES */

  .content-section {
    margin-top: 10px;
  }

  .user-search {
    width: 100%;
    max-width: 420px;
    padding: 12px 15px;
    margin-bottom: 22px;
    border: 1px solid #393244;
    border-radius: 8px;
    background: #14131b;
    color: white;
    font-size: 12px;
    outline: none;
  }

  .user-search:focus {
    border-color: #a78bfa;
  }

  .user-search::placeholder {
    color: #777184;
  }

  .table-wrapper {
    overflow-x: auto;
    border: 1px solid #2b2735;
    border-radius: 12px;
    background: #14131b;
  }

  table {
    width: 100%;
    min-width: 700px;
    border-collapse: collapse;
    text-align: left;
  }

  th {
    padding: 15px;
    background: #1b1923;
    color: #a9a1b8;
    font-size: 10px;
    text-transform: uppercase;
    letter-spacing: 0.8px;
  }

  td {
    padding: 15px;
    color: #ddd8e8;
    border-bottom: 1px solid #292532;
    font-size: 12px;
  }

  tbody tr:hover {
    background: #1d1928;
  }

  tbody tr:last-child td {
    border-bottom: none;
  }

  td a {
    color: #b69aff;
    text-decoration: none;
  }

  td a:hover {
    text-decoration: underline;
  }

  .user-cell {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .user-cell img,
  .user-initial {
    width: 31px;
    height: 31px;
    flex-shrink: 0;
    border-radius: 50%;
    object-fit: cover;
  }

  .user-initial {
    display: grid;
    place-items: center;
    background: #302343;
    color: #d4c3ff;
    font-weight: 700;
  }

  .user-info {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    gap: 4px;
  }

  .admin-user-tag {
    display: inline-block;
    padding: 3px 7px;
    border-radius: 5px;
    background: #33234d;
    color: #c4a7ff;
    font-size: 9px;
    font-weight: 800;
    letter-spacing: 0.8px;
  }

  .view-profile {
    white-space: nowrap;
  }

  .muted {
    color: #777184;
  }

  /* REPOSITORIES */

  .repository-list {
    display: grid;
    gap: 13px;
  }

  .repository-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 20px;
    padding: 20px;
    background: #14131b;
    border: 1px solid #2b2735;
    border-radius: 12px;
  }

  .repository-item:hover {
    border-color: #60438c;
  }

  .repo-details h3 {
    font-size: 15px;
    margin: 0 0 7px;
  }

  .repo-details p {
    color: #928a9f;
    font-size: 12px;
    margin: 0 0 10px;
  }

  .repo-meta {
    display: flex;
    flex-wrap: wrap;
    gap: 12px;
    color: #aaa0b9;
    font-size: 10px;
  }

  .repo-meta span:first-child {
    color: #c4b5fd;
    background: #2b2140;
    padding: 4px 8px;
    border-radius: 5px;
  }

  .repo-actions {
    display: flex;
    align-items: center;
    gap: 13px;
    white-space: nowrap;
  }

  .repo-actions a {
    color: #b69aff;
    font-size: 12px;
    text-decoration: none;
  }

  .delete-button {
    background: #341e27;
    color: #fb9cac;
    border: 1px solid #693849;
    border-radius: 7px;
    padding: 9px 12px;
    font-size: 11px;
    cursor: pointer;
  }

  .delete-button:hover:not(:disabled) {
    background: #512534;
  }

  .delete-button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  /* COMMUNITY */

  .community-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(210px, 1fr));
    gap: 15px;
  }

  .community-card {
    background: #14131b;
    border: 1px solid #2b2735;
    border-radius: 12px;
    padding: 22px;
    text-align: center;
  }

  .community-card img,
  .community-avatar {
    width: 55px;
    height: 55px;
    border-radius: 50%;
    object-fit: cover;
    margin: 0 auto 12px;
  }

  .community-avatar {
    display: grid;
    place-items: center;
    background: #302343;
    color: #d4c3ff;
    font-size: 20px;
  }

  .community-card h3 {
    font-size: 14px;
    margin: 0 0 5px;
  }

  .community-card p {
    color: #898194;
    font-size: 11px;
    margin: 0 0 12px;
  }

  .community-card a {
    color: #b69aff;
    font-size: 12px;
    text-decoration: none;
  }

  /* OTHER STATES */

  .empty {
    padding: 35px;
    color: #90889e;
    background: #14131b;
    border: 1px dashed #393244;
    border-radius: 12px;
    text-align: center;
    font-size: 13px;
  }

  .error-message {
    color: #ff9eac;
    background: #301b24;
    border: 1px solid #66303f;
    padding: 13px;
    border-radius: 8px;
    margin-bottom: 20px;
    font-size: 12px;
  }

  .admin-footer {
    display: flex;
    justify-content: space-between;
    gap: 12px;
    border-top: 1px solid #292432;
    margin-top: 60px;
    padding-top: 20px;
    color: #716b7d;
    font-size: 10px;
  }

  .loading-screen,
  .access-denied {
    min-height: 80vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: #0b0b10;
    color: white;
    text-align: center;
  }

  .loading-screen p,
  .access-denied p {
    color: #938a9f;
  }

  .loader {
    width: 35px;
    height: 35px;
    border: 3px solid #332747;
    border-top-color: #a78bfa;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
  }

  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }

  .lock {
    font-size: 45px;
  }

  .access-denied a {
    color: #b69aff;
  }

  /* RESPONSIVE */

  @media (max-width: 1050px) {
    .stats-grid {
      grid-template-columns: repeat(2, 1fr);
    }

    .admin-main {
      padding: 28px 22px;
    }
  }

  @media (max-width: 750px) {
    .admin-layout {
      flex-direction: column;
    }

    .sidebar {
      width: auto;
      min-height: auto;
      padding: 15px;
      border-right: none;
      border-bottom: 1px solid #292432;
    }

    .brand {
      margin-bottom: 18px;
    }

    .sidebar-label,
    .sidebar-bottom {
      display: none;
    }

    .sidebar nav {
      display: flex;
      overflow-x: auto;
      gap: 7px;
    }

    .sidebar nav button {
      width: auto;
      white-space: nowrap;
      padding: 10px;
    }

    .active-dot {
      display: none;
    }

    .admin-main {
      padding: 25px 16px;
    }

    .quick-grid {
      grid-template-columns: 1fr;
    }
  }

  @media (max-width: 520px) {
    .topbar {
      align-items: flex-start;
      flex-direction: column;
    }

    .topbar h1 {
      font-size: 23px;
    }

    .stats-grid {
      gap: 9px;
    }

    .stat-card {
      padding: 15px;
    }

    .stat-card h2 {
      font-size: 27px;
    }

    .welcome-panel {
      padding: 20px;
    }

    .welcome-panel h2 {
      font-size: 19px;
    }

    .welcome-symbol {
      font-size: 45px;
    }

    .repository-item {
      align-items: flex-start;
      flex-direction: column;
    }

    .admin-footer {
      flex-direction: column;
    }
  }
</style>
