
<script>
  import { onMount } from 'svelte';
  import { supabase } from '../../lib/supabaseClient.js';

  let user = $state(null);
  let profile = $state(null);
  let repositories = $state([]);

  let loading = $state(true);
  let editing = $state(false);
  let saving = $state(false);
  let uploadingAvatar = $state(false);
  let error = $state('');
  let success = $state('');

  let fullName = $state('');
  let username = $state('');
  let bio = $state('');
  let skillsText = $state('');
  let avatarUrl = $state('');
  let avatarPreview = $state('');

  let isOwner = $state(false);

  let displayName = $derived(
    profile?.full_name || user?.user_metadata?.full_name || 'Developer'
  );

  let initials = $derived(
    displayName
      .split(' ')
      .filter(Boolean)
      .slice(0, 2)
      .map((word) => word[0].toUpperCase())
      .join('')
  );

  onMount(async () => {
    const { data: { user } } = await supabase.auth.getUser();

if (user) {
  const githubIdentity = user.identities?.find(
    (identity) => identity.provider === 'github'
  );

  if (githubIdentity) {
    const githubUsername =
      githubIdentity.identity_data?.user_name ||
      githubIdentity.identity_data?.preferred_username ||
      githubIdentity.identity_data?.login;

    if (githubUsername) {
      await supabase
        .from('profiles')
        .update({ github_username: githubUsername })
        .eq('id', user.id);
    }
  }
}
    await loadProfile();
  });

  async function loadProfile() {
    loading = true;
    error = '';

    try {
      const {
        data: { user: currentUser },
        error: authError
      } = await supabase.auth.getUser();

      if (authError) throw authError;

      user = currentUser;

      // Determine whose profile to display
      const params = new URLSearchParams(window.location.search);
      const requestedId = params.get('id');

      const profileId = requestedId || currentUser?.id;

      if (!profileId) {
        window.location.href = '/login?redirect=/profile';
        return;
      }

      isOwner = Boolean(currentUser && currentUser.id === profileId);

      // Fetch profile
      const { data: profileData, error: profileError } = await supabase
        .from('profiles')
        .select('*')
        .eq('id', profileId)
        .maybeSingle();

      if (profileError) {
        error = profileError.message;
        return;
      }

      if (!profileData) {
        error = 'This developer profile could not be found.';
        return;
      }

      profile = profileData;

      fullName = profileData.full_name || '';
      username = profileData.username || '';
      bio = profileData.bio || '';
      skillsText = (profileData.skills || []).join(', ');
      avatarUrl = profileData.avatar_url || '';
      avatarPreview = profileData.avatar_url || '';

      // Fetch this developer's public repositories
      const { data: repoData, error: repoError } = await supabase
        .from('repositories')
        .select('*')
        .eq('user_id', profileId)
        .order('created_at', { ascending: false });

      if (repoError) {
        error = repoError.message;
        return;
      }

      repositories = repoData || [];

    } catch (err) {
      error = err.message || 'Something went wrong while loading this profile.';
    } finally {
      loading = false;
    }
  }

  // Upload profile picture to Supabase Storage
  async function uploadAvatar(event) {
    const input = event.currentTarget;
    const file = input.files?.[0];

    if (!file) return;

    error = '';
    success = '';

    if (!isOwner || !user) {
      error = 'You must be logged in to upload a profile picture.';
      input.value = '';
      return;
    }

    const allowedTypes = [
      'image/jpeg',
      'image/png',
      'image/webp',
      'image/gif'
    ];

    if (!allowedTypes.includes(file.type)) {
      error = 'Please choose a JPG, PNG, WEBP, or GIF image.';
      input.value = '';
      return;
    }

    if (file.size > 5 * 1024 * 1024) {
      error = 'Image size must be less than 5 MB.';
      input.value = '';
      return;
    }

    uploadingAvatar = true;

    try {
      const extensions = {
        'image/jpeg': 'jpg',
        'image/png': 'png',
        'image/webp': 'webp',
        'image/gif': 'gif'
      };

      const extension = extensions[file.type];
      const filePath = `${user.id}/${crypto.randomUUID()}.${extension}`;

      const { error: uploadError } = await supabase.storage
        .from('avatars')
        .upload(filePath, file, {
          cacheControl: '3600',
          upsert: false,
          contentType: file.type
        });

      if (uploadError) {
        throw uploadError;
      }

      const { data } = supabase.storage
        .from('avatars')
        .getPublicUrl(filePath);

      avatarUrl = data.publicUrl;
      avatarPreview = data.publicUrl;

      success = 'Photo uploaded! Click Save Changes to update your profile.';

    } catch (err) {
      error = err.message || 'Could not upload your photo.';
    } finally {
      uploadingAvatar = false;
      input.value = '';
    }
  }

  async function saveProfile() {
    if (!isOwner || !user) return;

    error = '';
    success = '';

    const cleanName = fullName.trim();
    const cleanUsername = username.trim().toLowerCase();

    if (!cleanName) {
      error = 'Please enter your full name.';
      return;
    }

    if (!/^[a-z0-9_]{3,20}$/.test(cleanUsername)) {
      error = 'Username must be 3–20 characters using letters, numbers or underscores.';
      return;
    }

    saving = true;

    try {
      const skills = skillsText
        .split(',')
        .map((skill) => skill.trim())
        .filter(Boolean);

      const { data, error: updateError } = await supabase
        .from('profiles')
        .update({
          full_name: cleanName,
          username: cleanUsername,
          bio: bio.trim(),
          skills,
          avatar_url: avatarUrl.trim() || null
        })
        .eq('id', user.id)
        .select()
        .single();

      if (updateError) {
        if (updateError.code === '23505') {
          error = 'That username is already taken.';
        } else {
          error = updateError.message;
        }
        return;
      }

      profile = data;
      avatarPreview = data.avatar_url || '';
      editing = false;
      success = 'Your profile has been updated!';

    } catch (err) {
      error = err.message || 'Could not save your profile. Please try again.';
    } finally {
      saving = false;
    }
  }
  async function connectGitHub() {
  const { data: { user }, error: userError } =
    await supabase.auth.getUser();

  if (userError || !user) {
    window.location.href = '/login?redirect=/profile';
    return;
  }

  const { error } = await supabase.auth.linkIdentity({
    provider: 'github',
    options: {
      redirectTo: `${window.location.origin}/profile`
    }
  });

  if (error) {
    alert(error.message);
  }
}
</script>

<svelte:head>
  <title>{profile?.full_name || 'Developer'} | DevSpace</title>
</svelte:head>

<main class="profile-page">

  {#if loading}
    <div class="status">Loading developer profile...</div>

  {:else if error && !profile}
    <div class="status error">{error}</div>

  {:else if profile}
    <section class="profile-header">

      <div class="profile-main">
        {#if profile.avatar_url}
          <img
            src={profile.avatar_url}
            alt="Profile picture"
            class="avatar"
          />
        {:else}
          <div class="avatar initials">{initials}</div>
        {/if}

        <div class="identity">
          <span class="eyebrow">DEVELOPER PROFILE</span>
          <h1>{profile.full_name}</h1>
          <p class="username">@{profile.username || 'username-not-set'}</p>
        </div>
      </div>

      {#if isOwner}
        <button class="edit-button" onclick={() => editing = !editing}>
          {editing ? 'Cancel Editing' : 'Edit Profile'}
        </button>
{#if profile?.github_username}
  <div class="github-connected">
    <span>✓ GitHub Connected</span>
    <a
      href={`https://github.com/${encodeURIComponent(profile.github_username)}`}
      target="_blank"
      rel="noopener noreferrer"
    >
      @{profile.github_username}
    </a>
  </div>
{:else}
  <button onclick={connectGitHub}>
    Connect GitHub
  </button>
{/if}
      {/if}

    </section>

    {#if error}
      <p class="message error">{error}</p>
    {/if}

    {#if success}
      <p class="message success">{success}</p>
    {/if}

    {#if editing && isOwner}
      <section class="edit-card">
        <h2>Edit your profile</h2>

        <form onsubmit={(event) => {
          event.preventDefault();
          saveProfile();
        }}>
          <label for="fullName">Full name</label>
          <input id="fullName" bind:value={fullName} required />

          <label for="username">Username</label>
          <input
            id="username"
            bind:value={username}
            placeholder="your_username"
            required
          />

          <label for="bio">About me</label>
          <textarea
            id="bio"
            bind:value={bio}
            rows="4"
            placeholder="Tell the community about yourself..."
          ></textarea>

          <label for="skills">Skills</label>
          <input
            id="skills"
            bind:value={skillsText}
            placeholder="Java, JavaScript, Python, Svelte"
          />
          <p class="hint">Separate skills with commas.</p>

          <label for="avatarUpload">Profile picture</label>

          <div class="avatar-upload">
            {#if avatarPreview}
              <img
                src={avatarPreview}
                alt="Profile picture preview"
                class="avatar-preview"
              />
            {:else}
              <div class="avatar-preview initials">{initials}</div>
            {/if}

            <div class="upload-controls">
              <input
                id="avatarUpload"
                type="file"
                accept="image/jpeg,image/png,image/webp,image/gif"
                onchange={uploadAvatar}
                disabled={uploadingAvatar}
              />

              <p class="hint">
                JPG, PNG, WEBP or GIF. Maximum size: 5 MB.
              </p>

              {#if uploadingAvatar}
                <p class="upload-status">Uploading your photo...</p>
              {/if}
            </div>
          </div>

          <button
            class="save-button"
            type="submit"
            disabled={saving || uploadingAvatar}
          >
            {saving ? 'Saving...' : 'Save Changes'}
          </button>
        </form>
      </section>

    {:else}
      <section class="info-grid">

        <article class="info-card">
          <h2>About me</h2>
          <p>
            {profile.bio || 'This developer has not added a bio yet.'}
          </p>
        </article>

        <article class="info-card">
          <h2>Skills & technologies</h2>

          {#if profile.skills?.length}
            <div class="skills">
              {#each profile.skills as skill}
                <span class="skill">{skill}</span>
              {/each}
            </div>
          {:else}
            <p>No skills added yet.</p>
          {/if}
        </article>

      </section>
    {/if}

    <section class="repositories-section">
      <div class="section-heading">
        <div>
          <span class="eyebrow">WHAT I'VE BUILT</span>
          <h2>My Repositories</h2>
        </div>
        <span class="repo-count">{repositories.length} projects</span>
      </div>

      {#if repositories.length === 0}
        <div class="empty-state">
          <h3>No repositories yet</h3>
          <p>This developer hasn't published any repositories.</p>
        </div>
      {:else}
        <div class="repository-grid">
          {#each repositories as repo (repo.id)}
            <article class="repository-card">
              <span class="category">{repo.category || 'Other'}</span>

              <h3>{repo.name}</h3>

              <p>
                {repo.description || 'No description provided.'}
              </p>

              <a
                href={repo.github_url}
                target="_blank"
                rel="noopener noreferrer"
              >
                View on GitHub ↗
              </a>
            </article>
          {/each}
        </div>
      {/if}
    </section>

  {/if}
</main>

<style>
  .profile-page {
    max-width: 1050px;
    margin: auto;
    padding: 65px 6% 90px;
    color: #26364a;
    min-height: 75vh;
  }

  .status {
    text-align: center;
    padding: 70px 20px;
    color: #778397;
  }

  .error {
    color: #c03939;
  }

  .profile-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 25px;
    flex-wrap: wrap;
    margin-bottom: 35px;
  }

  .profile-main {
    display: flex;
    align-items: center;
    gap: 22px;
  }

  .avatar {
    width: 105px;
    height: 105px;
    border-radius: 50%;
    object-fit: cover;
    flex-shrink: 0;
  }

  .initials {
    display: flex;
    align-items: center;
    justify-content: center;
    background: #5288d9;
    color: white;
    font-size: 32px;
    font-weight: 700;
  }

  .eyebrow {
    color: #5288d9;
    font-size: 11px;
    font-weight: 700;
    letter-spacing: 2px;
  }

  .identity h1 {
    font-size: 32px;
    margin: 8px 0;
  }

  .username {
    color: #778397;
    margin: 0;
  }

  .edit-button,
  .save-button {
    border: none;
    border-radius: 8px;
    padding: 12px 20px;
    background: #5288d9;
    color: white;
    font-weight: 600;
    cursor: pointer;
  }

  .info-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 22px;
    margin-bottom: 55px;
  }

  .info-card,
  .edit-card,
  .repository-card,
  .empty-state {
    border: 1px solid #e8edf4;
    border-radius: 14px;
    padding: 25px;
    background: white;
  }

  .info-card h2,
  .edit-card h2 {
    font-size: 19px;
    margin: 0 0 18px;
  }

  .info-card p,
  .repository-card p,
  .empty-state p {
    color: #778397;
    line-height: 1.7;
    font-size: 14px;
  }

  .skills {
    display: flex;
    flex-wrap: wrap;
    gap: 9px;
  }

  .skill,
  .category {
    display: inline-block;
    background: #edf4ff;
    color: #5288d9;
    border-radius: 20px;
    padding: 7px 12px;
    font-size: 12px;
    font-weight: 600;
  }

  .edit-card {
    max-width: 650px;
    margin-bottom: 55px;
    background: #f9fbfe;
  }

  .edit-card form {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .edit-card label {
    font-size: 13px;
    font-weight: 600;
    margin-top: 8px;
  }

  .edit-card input,
  .edit-card textarea {
    width: 100%;
    box-sizing: border-box;
    padding: 12px;
    border: 1px solid #dce3ec;
    border-radius: 8px;
    font: inherit;
    background: white;
    color: #26364a;
  }

  .avatar-upload {
    display: flex;
    align-items: center;
    gap: 20px;
    flex-wrap: wrap;
    margin: 8px 0 15px;
  }

  .avatar-preview {
    width: 90px;
    height: 90px;
    border-radius: 50%;
    object-fit: cover;
    flex-shrink: 0;
  }

  .upload-controls {
    display: flex;
    flex-direction: column;
    gap: 8px;
    min-width: 0;
    flex: 1;
  }

  .upload-controls input {
    max-width: 100%;
  }

  .hint {
    margin: 0;
    font-size: 12px;
    color: #929cac;
  }

  .upload-status {
    margin: 0;
    color: #5288d9;
    font-size: 13px;
  }

  .save-button {
    margin-top: 15px;
  }

  .save-button:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }

  .message {
    padding: 12px;
    border-radius: 8px;
    margin-bottom: 20px;
    font-size: 14px;
  }

  .message.error {
    background: #fff0f0;
  }

  .message.success {
    background: #edf9f1;
    color: #218653;
  }

  .section-heading {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 15px;
    margin-bottom: 25px;
  }

  .section-heading h2 {
    margin: 8px 0 0;
    font-size: 26px;
  }

  .repo-count {
    color: #778397;
    font-size: 13px;
  }

  .repository-grid {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 20px;
  }

  .repository-card h3 {
    margin: 18px 0 8px;
    overflow-wrap: anywhere;
  }

  .repository-card a {
    color: #5288d9;
    text-decoration: none;
    font-size: 13px;
    font-weight: 600;
  }

  .empty-state {
    text-align: center;
    background: #f9fbfe;
  }

  @media (max-width: 650px) {
    .info-grid,
    .repository-grid {
      grid-template-columns: 1fr;
    }

    .profile-header {
      align-items: flex-start;
    }

    .identity h1 {
      font-size: 25px;
    }

    .avatar {
      width: 80px;
      height: 80px;
    }
  }
  .github-connected {
  display: flex;
  flex-direction: column;
  gap: 6px;
  margin-top: 10px;
  font-size: 13px;
}

.github-connected span {
  color: #22c55e;
  font-weight: 600;
}

.github-connected a {
  color: #a78bfa;
  text-decoration: none;
}
</style>
