
<script>
  // Homepage logic will go here later
</script>

<svelte:head>
  <title>DevSpace — GitHub Repository Showcase</title>
  <meta
    name="description"
    content="Explore GitHub repositories in a clean developer dashboard."
  />
</svelte:head>

<div class="app-container">

  <!-- NAVBAR -->
  <nav class="navbar">
    <div class="logo">
      <span class="logo-icon">⌘</span>
      <span>devspace<span class="logo-dot">.</span></span>
    </div>

    <div class="nav-links">
  <a href="#home">Home</a>
  <a href="#explore">Explore</a>
  <a href="#my-repositories">My Repositories</a>
  <a href="#community">Community</a>
</div>

    <a href="#my-repositories" class="github-button">
  Connect GitHub ↗
</a>

  </nav>


  <!-- HERO SECTION -->
  <section class="hero" id="home">

    <div class="hero-label">
      <span class="status-dot"></span>
      YOUR DEVELOPER SPACE
    </div>

    <h1>
      Your code.<br />
      Your <span>journey.</span>
    </h1>

    <p class="hero-description">
      A beautiful space to explore, organize and showcase
      your GitHub repositories.
    </p>

    <div class="hero-actions">
      <a href="#repositories" class="primary-button">
        Explore repositories →
      </a>

      <a href="https://github.com" target="_blank" rel="noreferrer" class="secondary-button">
        Visit GitHub
      </a>
    </div>

  </section>


  

<!-- EXPLORE SECTION -->

<section class="repositories-section" id="explore">

  <div class="section-heading">
    <div>
      <p class="section-label">DISCOVER SOMETHING NEW</p>
      <h2>Explore repositories</h2>
      <p class="section-description">
        Find interesting projects built by developers around the world.
      </p>
    </div>
  </div>

  <div class="search-container">
    <span>⌕</span>
    <input
      type="text"
      placeholder="Search repositories, projects or languages..."
    />
  </div>

  <div class="filter-row">
    <button class="filter active">All projects</button>
    <button class="filter">JavaScript</button>
    <button class="filter">Python</button>
    <button class="filter">Java</button>
    <button class="filter">C++</button>
  </div>

  <div class="empty-state">
    <div class="empty-icon">⌘</div>
    <h3>Discover your next favorite project.</h3>
    <p>
      Public GitHub repositories will appear here.
      Start exploring once we connect the GitHub API.
    </p>
  </div>

</section>


<!-- MY REPOSITORIES SECTION -->

<section class="my-repositories" id="my-repositories">

  <p class="section-label">YOUR WORKSPACE</p>
  <h2>Your repositories</h2>

  <p class="section-description">
    Connect your GitHub account to bring your projects into DevSpace.
  </p>

  <div class="connect-card">
    <div class="connect-icon">
      <span>⌘</span>
    </div>

    <h3>Your GitHub journey starts here.</h3>

    <p>
      Connect your account to explore your repositories,
      manage your projects and choose what to feature.
    </p>

    <a href="https://github.com" target="_blank"
       rel="noreferrer" class="primary-button">
      Connect with GitHub →
    </a>
  </div>

</section>


<!-- COMMUNITY SECTION -->

<section class="community-section" id="community">

  <p class="section-label">BUILT BY DEVELOPERS</p>
  <h2>From the community.</h2>

  <p class="section-description">
    Explore projects featured by developers who use DevSpace.
  </p>

  <div class="empty-state">
    <h3>Great projects deserve to be discovered.</h3>
    <p>
      Community-featured repositories will appear here.
    </p>
  </div>

</section>


</div>
<style>

.app-container {
  max-width: 1150px;
  margin: auto;
  padding: 0 35px;
}

/* NAVBAR */

.navbar {
  height: 85px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid var(--border);
}

.logo {
  display: flex;
  align-items: center;
  gap: 9px;
  font-size: 22px;
  font-weight: 750;
  letter-spacing: -1px;
}

.logo-icon {
  background: var(--primary);
  color: var(--text);
  padding: 6px 10px;
  border-radius: 9px;
}

.logo-dot {
  color: var(--primary-dark);
}

.nav-links {
  display: flex;
  gap: 32px;
  color: var(--muted);
  font-size: 14px;
}

.nav-links a:hover {
  color: var(--primary-dark);
}

.github-button {
  padding: 10px 18px;
  border: 1px solid var(--border);
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
}

/* HERO */

.hero {
  padding: 105px 0 90px;
}

.hero-label,
.section-label {
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 2px;
  color: var(--primary-dark);
}

.status-dot {
  display: inline-block;
  height: 7px;
  width: 7px;
  border-radius: 50%;
  background: #78b89a;
  margin-right: 7px;
}

.hero h1 {
  font-size: clamp(48px, 7vw, 76px);
  line-height: 1.08;
  letter-spacing: -4px;
  margin-top: 25px;
  font-weight: 750;
}

.hero h1 span {
  color: var(--primary-dark);
}

.hero-description {
  max-width: 460px;
  color: var(--muted);
  line-height: 1.8;
  margin-top: 22px;
  font-size: 16px;
}

.hero-actions {
  display: flex;
  gap: 14px;
  margin-top: 30px;
}

.primary-button,
.secondary-button {
  padding: 13px 20px;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
}

.primary-button {
  background: var(--text);
  color: white;
}

.primary-button:hover {
  background: var(--primary-dark);
}

.secondary-button {
  border: 1px solid var(--border);
}

/* PROFILE */

.profile-section {
  padding: 45px 0;
  border-top: 1px solid var(--border);
}

.profile-heading {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 22px;
}

.profile-heading h2,
.repositories-section h2 {
  font-size: 25px;
  margin-top: 9px;
  letter-spacing: -0.8px;
}

.profile-badge {
  font-size: 12px;
  color: var(--primary-dark);
  background: #eaf2fa;
  padding: 8px 12px;
  border-radius: 20px;
}

.profile-card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 28px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 25px;
}

.profile-info {
  display: flex;
  align-items: center;
  gap: 17px;
}

.avatar-placeholder {
  width: 65px;
  height: 65px;
  border-radius: 50%;
  background: #dceaf7;
  display: grid;
  place-items: center;
  color: var(--primary-dark);
  font-size: 20px;
  font-weight: 700;
}

.profile-info h3 {
  font-size: 17px;
}

.profile-info p {
  color: var(--muted);
  font-size: 13px;
  margin-top: 4px;
}

.profile-info span {
  display: block;
  font-size: 12px;
  color: var(--muted);
  margin-top: 8px;
}

.profile-stats {
  display: flex;
  gap: 35px;
}

.stat {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.stat strong {
  font-size: 21px;
}

.stat span {
  font-size: 12px;
  color: var(--muted);
}

/* REPOSITORIES */

.repositories-section {
  padding: 65px 0;
  border-top: 1px solid var(--border);
}

.section-description {
  color: var(--muted);
  margin-top: 10px;
  font-size: 14px;
}

.empty-state {
  margin-top: 25px;
  padding: 55px 20px;
  border: 1px dashed #cbd8e5;
  border-radius: 12px;
  text-align: center;
  color: var(--muted);
  font-size: 14px;
}

/* RESPONSIVE */

@media (max-width: 700px) {
  .app-container {
    padding: 0 20px;
  }

  .nav-links {
    display: none;
  }

  .hero {
    padding: 75px 0;
  }

  .hero h1 {
    letter-spacing: -2px;
  }

  .profile-card {
    flex-direction: column;
    align-items: flex-start;
  }

  .profile-stats {
    width: 100%;
    justify-content: space-between;
    gap: 10px;
  }
}

@media (max-width: 420px) {
  .logo {
    font-size: 18px;
  }

  .github-button {
    padding: 9px 12px;
  }

  .profile-info {
    gap: 10px;
  }

  .avatar-placeholder {
    width: 52px;
    height: 52px;
  }

  .hero-actions {
    flex-direction: column;
    align-items: flex-start;
  }

}

.submit-section {
  padding: 55px 0;
  border-top: 1px solid var(--border);
}

.submit-section h2 {
  font-size: 28px;
  margin-top: 12px;
}

.submit-section .primary-button {
  display: inline-block;
  margin-top: 22px;
}

/* SECTION HEADINGS */

.section-heading {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* SEARCH */

.search-container {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-top: 30px;
  padding: 15px 18px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 10px;
  color: var(--muted);
}

.search-container input {
  width: 100%;
  outline: none;
  border: none;
  background: transparent;
  color: var(--text);
  font-size: 14px;
}

/* FILTERS */

.filter-row {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 20px;
}

.filter {
  background: var(--surface);
  border: 1px solid var(--border);
  padding: 9px 15px;
  border-radius: 20px;
  color: var(--muted);
  font-size: 13px;
}

.filter.active {
  background: #eaf2fa;
  color: var(--primary-dark);
  border-color: #d5e5f5;
}

/* EMPTY STATE */

.empty-state {
  margin-top: 30px;
  padding: 65px 20px;
  background: var(--surface);
  border: 1px dashed #cbd8e5;
  border-radius: 14px;
  text-align: center;
}

.empty-icon,
.connect-icon {
  width: 55px;
  height: 55px;
  display: grid;
  place-items: center;
  background: #eaf2fa;
  color: var(--primary-dark);
  border-radius: 14px;
  font-size: 25px;
  margin: 0 auto 18px;
}

.empty-state h3,
.connect-card h3 {
  font-size: 19px;
  margin-bottom: 10px;
}

.empty-state p,
.connect-card p {
  max-width: 440px;
  margin: auto;
  color: var(--muted);
  font-size: 14px;
  line-height: 1.7;
}

/* MY REPOSITORIES */

.my-repositories,
.community-section {
  padding: 65px 0;
  border-top: 1px solid var(--border);
}

.my-repositories h2,
.community-section h2 {
  font-size: 26px;
  margin-top: 10px;
}

.connect-card {
  margin-top: 30px;
  padding: 50px 20px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 14px;
  text-align: center;
}

.connect-card .primary-button {
  display: inline-block;
  margin-top: 22px;
}

@media (max-width: 700px) {
  .search-container {
    padding: 13px;
  }

  .empty-state,
  .connect-card {
    padding: 40px 18px;
  }
}


</style>