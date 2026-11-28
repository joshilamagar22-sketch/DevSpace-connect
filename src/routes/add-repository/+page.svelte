
<script>
  import { supabase } from '../../lib/supabaseClient.js';
  import { onMount } from 'svelte';

  let isLoggedIn = $state(false);
  let checkingAuth = $state(true);

  let repositoryUrl = $state('');
  let description = $state('');
  let category = $state('Web Development');
  let error = $state('');
  let submittedRepository = $state(null);
  let publishing = $state(false);
let publishMessage = $state('');

  onMount(async () => {
    const { data: { session }, error: authError } =
      await supabase.auth.getSession();

    checkingAuth = false;

    if (authError || !session) {
      window.location.href = '/login?redirect=/add-repository';
      return;
    }

    isLoggedIn = true;
  });

  /** @param {SubmitEvent} event */
  function handleSubmit(event) {
    event.preventDefault();

    error = '';
    submittedRepository = null;

    try {
      const url = new URL(repositoryUrl.trim());
      const parts = url.pathname.split('/').filter(Boolean);

      if (
        !['github.com', 'www.github.com'].includes(url.hostname) ||
        parts.length < 2
      ) {
        error = 'Please enter a valid GitHub repository URL.';
        return;
      }

      const owner = parts[0];
      const name = parts[1].replace(/\.git$/, '');

      if (!owner || !name) {
        error = 'Please enter a valid GitHub repository URL.';
        return;
      }

      submittedRepository = {
        owner,
        name,
        url: `https://github.com/${owner}/${name}`,
        description: description.trim(),
        category
      };

    } catch {
      error = 'Please enter a valid URL, such as https://github.com/username/project.';
    }
  }
  async function publishRepository() {
  error = '';
  publishMessage = '';
  publishing = true;

  try {
    const { data: { user }, error: userError } =
      await supabase.auth.getUser();

    if (userError || !user) {
      window.location.href = '/login?redirect=/add-repository';
      return;
    }

    const { error: insertError } = await supabase
      .from('repositories')
      .insert({
        user_id: user.id,
        name: submittedRepository.name,
        github_url: submittedRepository.url,
        description: submittedRepository.description,
        category: submittedRepository.category
      });

    if (insertError) {
      if (insertError.code === '23505') {
        error = 'You have already added this repository.';
      } else {
        error = insertError.message;
      }
      return;
    }

    publishMessage = 'Repository published successfully!';

  } catch (err) {
    error = 'Something went wrong. Please try again.';
  } finally {
    publishing = false;
  }
}
</script>

<svelte:head>
  <title>Add Repository | DevSpace</title>
</svelte:head>

{#if checkingAuth}
  <main class="add-page">
    <p class="loading-message">Checking your login...</p>
  </main>
{:else if isLoggedIn}
  <main class="add-page">
    <section class="page-heading">
      <span class="eyebrow">SHOWCASE YOUR WORK</span>
      <h1>Add Your Repository</h1>
      <p>Share your GitHub project and let other developers discover what you've built.</p>
    </section>

    <section class="form-card">
      <form onsubmit={handleSubmit}>
        <label for="repositoryUrl">GitHub Repository URL *</label>
        <input
          id="repositoryUrl"
          type="url"
          placeholder="https://github.com/username/project"
          bind:value={repositoryUrl}
          required
        />
        <p class="hint">Paste the link to your public GitHub repository.</p>

        <label for="description">Project Description</label>
        <textarea
          id="description"
          placeholder="What does your project do? What problem does it solve?"
          bind:value={description}
          rows="5"
        ></textarea>

        <label for="category">Project Category</label>
        <select id="category" bind:value={category}>
          <option>Web Development</option>
          <option>Mobile Development</option>
          <option>Artificial Intelligence</option>
          <option>Machine Learning</option>
          <option>Game Development</option>
          <option>IoT and Robotics</option>
          <option>Other</option>
        </select>

        <button type="submit" class="submit-button">
          Preview Repository →
        </button>

        {#if error}
          <p class="error-message">{error}</p>
        {/if}
      </form>

      {#if submittedRepository}
        <div class="preview">
          <span class="eyebrow">REPOSITORY PREVIEW</span>
          <h2>{submittedRepository.name}</h2>
          <p class="owner">By @{submittedRepository.owner}</p>

          <p>{submittedRepository.description || 'No description added yet.'}</p>

          <span class="category">{submittedRepository.category}</span>

          <a
            href={submittedRepository.url}
            target="_blank"
            rel="noreferrer"
          >
            Open GitHub Repository ↗
          </a>
          <button
  class="publish-button"
  onclick={publishRepository}
  disabled={publishing || !!publishMessage}
>
  {publishing ? 'Publishing...' : 'Publish to DevSpace →'}
</button>

{#if publishMessage}
  <p class="success-message">{publishMessage}</p>
{/if}

          <p class="notice">
            Your preview is ready! Publishing and saving repositories will be
            enabled when we connect the database.
          </p>
        </div>
      {/if}
    </section>
  </main>
{/if}

<style>
  .add-page {
    min-height: 75vh;
    padding: 65px 6% 90px;
    color: #26364a;
    background: #f9fbfe;
  }

  .loading-message {
    text-align: center;
    padding: 60px;
    color: #778397;
  }

  .page-heading {
    text-align: center;
    margin-bottom: 40px;
  }

  .eyebrow {
    color: #5288d9;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 2px;
  }

  .page-heading h1 {
    font-size: 38px;
    margin: 15px 0 10px;
  }

  .page-heading p {
    color: #778397;
    line-height: 1.7;
  }

  .form-card {
    max-width: 650px;
    margin: auto;
    padding: 35px;
    background: white;
    border: 1px solid #e8edf4;
    border-radius: 15px;
  }

  form {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  label {
    font-size: 13px;
    font-weight: 600;
    color: #26364a;
  }

  input, textarea, select {
    width: 100%;
    box-sizing: border-box;
    padding: 13px;
    border: 1px solid #dfe6ef;
    border-radius: 8px;
    font-family: inherit;
    font-size: 14px;
    color: #26364a;
    outline: none;
    background: white;
  }

  input:focus, textarea:focus, select:focus {
    border-color: #5288d9;
  }

  textarea {
    resize: vertical;
  }

  .hint {
    color: #8994a5;
    font-size: 12px;
    margin-top: -5px;
  }

  .submit-button {
    margin-top: 12px;
    padding: 14px;
    background: #5288d9;
    color: white;
    border: none;
    border-radius: 8px;
    font-weight: 600;
    cursor: pointer;
  }

  .submit-button:hover {
    background: #3f73c0;
  }

  .error-message {
    color: #c0392b;
    font-size: 13px;
  }

  .preview {
    margin-top: 35px;
    padding: 25px;
    border: 1px solid #dce8f8;
    border-radius: 12px;
    background: #f7faff;
  }

  .preview h2 {
    margin: 15px 0 5px;
    overflow-wrap: anywhere;
  }

  .preview p {
    color: #778397;
    line-height: 1.7;
  }

  .preview .owner {
    font-size: 13px;
  }

  .category {
    display: inline-block;
    padding: 6px 12px;
    margin: 8px 0;
    border-radius: 20px;
    background: #e8f1ff;
    color: #5288d9;
    font-size: 12px;
  }

  .preview a {
    display: block;
    margin-top: 15px;
    color: #5288d9;
    font-weight: 600;
    text-decoration: none;
  }

  .preview .notice {
    margin-top: 20px;
    padding-top: 15px;
    border-top: 1px solid #dce8f8;
    font-size: 12px;
  }

  @media (max-width: 600px) {
    .form-card {
      padding: 25px 18px;
    }

    .page-heading h1 {
      font-size: 32px;
    }
  }
  .publish-button {
  display: block;
  width: 100%;
  margin-top: 20px;
  padding: 14px;
  background: #26364a;
  color: white;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
}

.publish-button:hover {
  background: #3b506b;
}

.publish-button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.success-message {
  color: #218653;
  font-size: 13px;
  margin-top: 12px;
}
</style>
