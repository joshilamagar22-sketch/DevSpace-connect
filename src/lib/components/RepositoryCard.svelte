
<script>
  let { repo, onSelect } = $props();
</script>

<article class="repo-card">
  <div class="card-top">
    <span class="repo-icon">⌘</span>
    <span class="language">{repo.language || 'Unknown'}</span>
  </div>

  <h3>{repo.name}</h3>

  <p class="description">
    {repo.description || 'No description provided.'}
  </p>

  <div class="repo-stats">
    <span>☆ {repo.stars ?? 0} Stars</span>
    <span>⑂ {repo.forks ?? 0} Forks</span>
  </div>

  <div class="card-bottom">
    <span class="owner">@{repo.owner}</span>

    <button onclick={() => onSelect(repo)}>
      View details →
    </button>
  </div>
</article>

<style>
  .repo-card {
    background: white;
    border: 1px solid #e8edf4;
    border-radius: 14px;
    padding: 23px;
    transition: transform 0.2s, box-shadow 0.2s;
  }

  .repo-card:hover {
    transform: translateY(-4px);
    box-shadow: 0 10px 30px rgba(35, 55, 80, 0.08);
  }

  .card-top,
  .repo-stats,
  .card-bottom {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .repo-icon {
    font-size: 22px;
    color: #5288d9;
  }

  .language {
    background: #edf4ff;
    color: #5288d9;
    padding: 5px 10px;
    border-radius: 20px;
    font-size: 11px;
  }

  h3 {
    color: #26364a;
    margin: 22px 0 10px;
    font-size: 18px;
  }

  .description {
    color: #778397;
    font-size: 13px;
    line-height: 1.7;
    min-height: 44px;
  }

  .repo-stats {
    justify-content: flex-start;
    gap: 20px;
    margin: 20px 0;
    color: #778397;
    font-size: 12px;
  }

  .card-bottom {
    border-top: 1px solid #edf0f5;
    padding-top: 15px;
  }

  .owner {
    color: #8994a5;
    font-size: 12px;
  }

  button {
    border: none;
    background: transparent;
    color: #5288d9;
    font-weight: 600;
    cursor: pointer;
  }
</style>
