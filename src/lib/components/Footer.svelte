
<footer class="footer">
  <div class="footer-content">
    <div class="footer-brand">
      <a href="/" class="logo">Dev<span>Space</span></a>
      <p>A space to discover, share, and showcase developer projects.</p>
    </div>

    <div class="footer-links">
      <a href="/about">About</a>
      <a href="/explore">Explore</a>
      <a href="/community">Community</a>
    </div>
  </div>

  <div class="footer-bottom">
    <p>© 2026 DevSpace. Built for developers.</p>
  </div>
</footer>

<style>
  .footer {
    background: #f7f9fc;
    padding: 45px 6% 20px;
    border-top: 1px solid #e9edf3;
  }

  .footer-content {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    gap: 30px;
    flex-wrap: wrap;
  }

  .logo {
    font-size: 23px;
    font-weight: 800;
    color: #26364a;
    text-decoration: none;
  }

  .logo span {
    color: #5288d9;
  }

  .footer-brand p {
    color: #778397;
    font-size: 14px;
    max-width: 300px;
    line-height: 1.7;
  }

  .footer-links {
    display: flex;
    gap: 25px;
    flex-wrap: wrap;
  }

  .footer-links a {
    color: #526174;
    text-decoration: none;
    font-size: 14px;
  }

  .footer-links a:hover {
    color: #5288d9;
  }

  .footer-bottom {
    border-top: 1px solid #e1e7ef;
    margin-top: 30px;
    padding-top: 15px;
    color: #8994a5;
    font-size: 12px;
    text-align: center;
  }

  @media (max-width: 600px) {
    .footer-content {
      flex-direction: column;
    }
  }
</style>
