
<script>
  import RepositoryCard from '../../lib/components/RepositoryCard.svelte';

  let searchTerm = $state('');
  let selectedLanguage = $state('All');
  let selectedRepository = $state(null);

  // Temporary demo data. Later, we'll fetch real GitHub repositories.
  let repositories = [
    {
      id: 1,
      name: 'campus-connect',
      description: 'A platform connecting students with campus events and clubs.',
      language: 'JavaScript',
      stars: 24,
      forks: 6,
      owner: 'developer'
    },
    {
      id: 2,
      name: 'weather-app',
      description: 'A weather application displaying current weather conditions.',
      language: 'Python',
      stars: 18,
      forks: 4,
      owner: 'developer'
    },
    {
      id: 3,
      name: 'student-management',
      description: 'A system for managing student records and academic information.',
      language: 'Java',
      stars: 12,
      forks: 3,
      owner: 'developer'
    },
    {
      id: 4,
      name: 'portfolio-website',
      description: 'A responsive personal portfolio for showcasing projects.',
      language: 'CSS',
      stars: 9,
      forks: 2,
      owner: 'developer'
    }
  ];

  let languages = ['All', 'JavaScript', 'Python', 'Java', 'CSS'];

  let filteredRepositories = $derived(
    repositories.filter((repo) => {
      const matchesSearch =
        repo.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
        repo.description.toLowerCase().includes(searchTerm.toLowerCase());

      const matchesLanguage =
        selectedLanguage === 'All' ||
        repo.language === selectedLanguage;

      return matchesSearch && matchesLanguage;
    })
  );

  function closePopup() {
    selectedRepository = null;
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
        placeholder="Search repositories..."
        bind:value={searchTerm}
      />

      <select bind:value={selectedLanguage}>
        {#each languages as language}
          <option value={language}>{language}</option>
        {/each}
      </select>
    </div>

    <div class="results-heading">
      <h2>Discover repositories</h2>
      <span>{filteredRepositories.length} projects found</span>
    </div>

    <div class="repo-grid">
      {#each filteredRepositories as repo (repo.id)}
        <RepositoryCard
          {repo}
          onSelect={(selected) => selectedRepository = selected}
        />
      {/each}
    </div>

    {#if filteredRepositories.length === 0}
      <p class="empty-state">No repositories found. Try another search.</p>
    {/if}
  </section>
</main>

{#if selectedRepository}
  <div class="modal-overlay" onclick={closePopup}>
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
      <p>{selectedRepository.description}</p>
      <p>Owner: @{selectedRepository.owner}</p>
      <p>Language: {selectedRepository.language}</p>
      <p>Stars: ☆ {selectedRepository.stars}</p>
      <p>Forks: {selectedRepository.forks}</p>

      <a
        href={`https://github.com/${selectedRepository.owner}/${selectedRepository.name}`}
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

  .empty-state {
    text-align: center;
    padding: 45px;
    color: #778397;
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
