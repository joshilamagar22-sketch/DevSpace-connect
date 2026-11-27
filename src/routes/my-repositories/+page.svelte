
<script>
  let connected = $state(false);

  function connectGitHub() {
    // Temporary demo interaction.
    // We'll implement real GitHub OAuth later.
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

    <div class="connect-card">
      <div class="github-icon">⌘</div>

      {#if connected}
        <h2>GitHub connected!</h2>
        <p>This is currently a demo connection. Real GitHub authentication is coming next.</p>
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

  .connect-card {
    border: 1px solid #e8edf4;
    border-radius: 15px;
    padding: 55px 25px;
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

  @media (max-width: 600px) {
    .dashboard-heading {
      align-items: flex-start;
      flex-direction: column;
    }

    .page-heading h1 {
      font-size: 34px;
    }
  }
</style>
