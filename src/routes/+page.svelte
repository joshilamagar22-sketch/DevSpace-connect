
<script>
</script>

<svelte:head>
  <title>DevSpace — Discover Developer Projects</title>
  <meta
    name="description"
    content="Discover, share, and showcase developer projects on DevSpace."
  />
</svelte:head>

<main>
  <section class="hero">
    <div class="hero-content">
      <span class="eyebrow">A SPACE FOR DEVELOPERS</span>

      <h1>
        Your projects deserve
        <span>to be discovered.</span>
      </h1>

      <p>
        Explore innovative projects, share what you're building,
        and connect with developers who create.
      </p>

      <div class="hero-buttons">
        <a href="/explore" class="primary-button">
          Explore Projects →
        </a>

        <a href="/signup" class="secondary-button">
          Join DevSpace
        </a>
      </div>
    </div>
  </section>

  <section class="features">
    <div class="section-heading">
      <span class="eyebrow">WHAT YOU CAN DO</span>
      <h2>Everything your projects need</h2>
      <p>A place to discover, showcase, and grow.</p>
    </div>

    <div class="feature-grid">
      <article class="feature-card">
        <span class="feature-icon">⌕</span>
        <h3>Discover Projects</h3>
        <p>Explore interesting repositories and find inspiration from developers.</p>
      </article>

      <article class="feature-card">
        <span class="feature-icon">↗</span>
        <h3>Showcase Your Work</h3>
        <p>Add your repositories and let others discover what you've built.</p>
      </article>

      <article class="feature-card">
        <span class="feature-icon">◎</span>
        <h3>Join the Community</h3>
        <p>Share your projects and connect with fellow developers.</p>
      </article>
    </div>
  </section>
</main>

<style>
  main {
    color: #26364a;
  }

  .hero {
    min-height: 540px;
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding: 80px 6%;
    background: linear-gradient(135deg, #f7faff, #edf4ff);
  }

  .hero-content {
    max-width: 750px;
  }

  .eyebrow {
    color: #5288d9;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 2px;
  }

  h1 {
    font-size: clamp(38px, 6vw, 64px);
    line-height: 1.15;
    margin: 22px 0;
    letter-spacing: -2px;
  }

  h1 span {
    display: block;
    color: #5288d9;
  }

  .hero-content > p {
    color: #778397;
    font-size: 16px;
    line-height: 1.8;
    max-width: 580px;
    margin: 0 auto;
  }

  .hero-buttons {
    display: flex;
    justify-content: center;
    gap: 15px;
    flex-wrap: wrap;
    margin-top: 32px;
  }

  .primary-button,
  .secondary-button {
    padding: 13px 22px;
    border-radius: 8px;
    text-decoration: none;
    font-size: 14px;
    font-weight: 600;
  }

  .primary-button {
    background: #5288d9;
    color: white;
  }

  .secondary-button {
    background: white;
    color: #26364a;
    border: 1px solid #dce5f1;
  }

  .features {
    padding: 85px 6%;
  }

  .section-heading {
    text-align: center;
    margin-bottom: 45px;
  }

  .section-heading h2 {
    font-size: 32px;
    margin: 15px 0 10px;
  }

  .section-heading p {
    color: #778397;
  }

  .feature-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 22px;
    max-width: 1100px;
    margin: auto;
  }

  .feature-card {
    padding: 28px;
    border: 1px solid #e8edf4;
    border-radius: 14px;
    background: white;
  }

  .feature-icon {
    font-size: 28px;
    color: #5288d9;
  }

  .feature-card h3 {
    margin: 18px 0 10px;
  }

  .feature-card p {
    color: #778397;
    font-size: 14px;
    line-height: 1.7;
  }

  @media (max-width: 700px) {
    .feature-grid {
      grid-template-columns: 1fr;
    }

    .features {
      padding: 60px 6%;
    }
  }
</style>
