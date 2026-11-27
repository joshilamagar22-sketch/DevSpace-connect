
<svelte:head>
  <title>About DevSpace</title>
</svelte:head>

<main class="about-page">
  <section class="about-hero">
    <span class="eyebrow">ABOUT DE VSPACE</span>
    <h1>A place where developers and ideas connect.</h1>
    <p>
      DevSpace is a platform designed to help developers
      discover projects, showcase their work, and connect
      with a community of creators.
    </p>
  </section>

  <section class="mission">
    <div class="mission-content">
      <span class="eyebrow">OUR MISSION</span>
      <h2>Make great projects easier to discover.</h2>
      <p>
        Every developer builds something worth sharing.
        But finding the right audience and discovering
        inspiring work can be difficult.
      </p>
      <p>
        DevSpace brings projects and developers together
        in one place, making it easier to explore ideas,
        share creations, and learn from each other.
      </p>
    </div>
  </section>

  <section class="values">
    <div class="section-heading">
      <span class="eyebrow">WHAT WE BELIEVE IN</span>
      <h2>Built around developers</h2>
    </div>

    <div class="values-grid">
      <article class="value-card">
        <span>⌕</span>
        <h3>Discovery</h3>
        <p>Find interesting projects and get inspired by creative developers.</p>
      </article>

      <article class="value-card">
        <span>↗</span>
        <h3>Showcase</h3>
        <p>Give your projects a space where people can explore your work.</p>
      </article>

      <article class="value-card">
        <span>◎</span>
        <h3>Community</h3>
        <p>Connect with developers and learn from what others create.</p>
      </article>
    </div>
  </section>
</main>

<style>
  .about-page {
    color: #26364a;
    padding-bottom: 80px;
  }

  .about-hero {
    text-align: center;
    padding: 100px 8%;
    background: linear-gradient(135deg, #f7faff, #edf4ff);
  }

  .eyebrow {
    color: #5288d9;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 2px;
  }

  .about-hero h1 {
    max-width: 800px;
    margin: 22px auto;
    font-size: clamp(36px, 5vw, 55px);
    line-height: 1.2;
  }

  .about-hero p {
    max-width: 650px;
    margin: auto;
    color: #778397;
    line-height: 1.8;
  }

  .mission {
    max-width: 850px;
    margin: auto;
    padding: 85px 6%;
  }

  .mission h2 {
    font-size: 32px;
    margin: 18px 0;
  }

  .mission p {
    color: #778397;
    line-height: 1.9;
  }

  .values {
    padding: 30px 6%;
  }

  .section-heading {
    text-align: center;
    margin-bottom: 40px;
  }

  .section-heading h2 {
    font-size: 32px;
    margin-top: 15px;
  }

  .values-grid {
    max-width: 1100px;
    margin: auto;
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 22px;
  }

  .value-card {
    padding: 28px;
    border: 1px solid #e8edf4;
    border-radius: 14px;
  }

  .value-card > span {
    font-size: 28px;
    color: #5288d9;
  }

  .value-card h3 {
    margin: 18px 0 10px;
  }

  .value-card p {
    color: #778397;
    font-size: 14px;
    line-height: 1.8;
  }

  @media (max-width: 700px) {
    .values-grid {
      grid-template-columns: 1fr;
    }

    .about-hero {
      padding: 75px 6%;
    }
  }
</style>
