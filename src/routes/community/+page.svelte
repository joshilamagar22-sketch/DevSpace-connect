
<script>
  import RepositoryCard from '../../lib/components/RepositoryCard.svelte';

  let selectedRepository = $state(null);

  // Temporary community projects.
  // Later, these will come from Supabase.
  let communityProjects = [
    {
      id: 1,
      name: 'campus-connect',
      description: 'A platform for discovering campus events and student clubs.',
      language: 'JavaScript',
      stars: 24,
      forks: 6,
      owner: 'student-dev'
    },
    {
      id: 2,
      name: 'smart-attendance',
      description: 'An attendance management system using face recognition.',
      language: 'Python',
      stars: 31,
      forks: 8,
      owner: 'tech-creator'
    },
    {
      id: 3,
      name: 'expense-tracker',
      description: 'A simple application to track personal expenses.',
      language: 'Java',
      stars: 15,
      forks: 3,
      owner: 'code-builder'
    },
    {
      id: 4,
      name: 'portfolio-design',
      description: 'A clean and responsive portfolio website.',
      language: 'CSS',
      stars: 12,
      forks: 2,
      owner: 'design-dev'
    }
  ];

  function closePopup() {
    selectedRepository = null;
  }
</script>

<svelte:head>
  <title>Community | DevSpace</title>
</svelte:head>

<main class="community-page">
  <section class="page-heading">
    <span class="eyebrow">BUILT BY DEVELOPERS</span>
    <h1>Developer Community</h1>
    <p>
      Explore what others are building, discover new ideas,
      and share your own projects.
    </p>

  <a href="/add-repository" class="share-button">
  Share Your Project →
</a>
  </section>

  <section class="community-projects">
    <div class="section-heading">
      <div>
        <h2>Community Projects</h2>
        <p>Projects shared by developers on DevSpace.</p>
      </div>

      <span>{communityProjects.length} projects</span>
    </div>

    <div class="repo-grid">
      {#each communityProjects as repo (repo.id)}
        <RepositoryCard
          {repo}
          onSelect={(selected) => selectedRepository = selected}
        />
      {/each}
    </div>
  </section>
</main>

{#if selectedRepository}
  <div class="modal-overlay" onclick={closePopup}>
    <div
      class="modal-card"
      role="dialog"
      aria-modal="true"
      aria-labelledby="community-modal-title"
      onclick={(event) => event.stopPropagation()}
    >
      <button class="modal-close" onclick={closePopup}>×</button>

      <span class="eyebrow">COMMUNITY PROJECT</span>
      <h2 id="community-modal-title">{selectedRepository.name}</h2>
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
  .community-page {
    min-height: 75vh;
    padding: 65px 6% 90px;
    color: #26364a;
  }

  .page-heading {
    text-align: center;
    padding: 35px 0 65px;
  }

  .eyebrow {
    color: #5288d9;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 2px;
  }

  .page-heading h1 {
    font-size: 42px;
    margin: 15px 0;
  }

  .page-heading p {
    max-width: 550px;
    margin: auto;
    color: #778397;
    line-height: 1.8;
  }

  .share-button {
    display: inline-block;
    margin-top: 25px;
    padding: 12px 20px;
    background: #5288d9;
    color: white;
    text-decoration: none;
    border-radius: 8px;
    font-size: 14px;
    font-weight: 600;
  }

  .community-projects {
    max-width: 1150px;
    margin: auto;
  }

  .section-heading {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 15px;
    margin-bottom: 25px;
  }

  .section-heading h2 {
    margin-bottom: 7px;
  }

  .section-heading p,
  .section-heading > span {
    color: #778397;
    font-size: 13px;
  }

  .repo-grid {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 22px;
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

    .page-heading h1 {
      font-size: 34px;
    }
  }
</style>
