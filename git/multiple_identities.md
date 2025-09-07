# Seamless Git & SSH Configuration for Multiple Accounts 🚀

## 1. The Challenge: Managing Multiple Identities 🤔

As a developer, it's common to work with multiple Git hosting services and accounts simultaneously. For example:

* A personal GitHub account.
* A personal Bitbucket account.
* An organization's Bitbucket or GitHub account.

The challenge arises because when you use SSH, the remote URL (e.g., `git@github.com:...`) doesn't inherently know which of your identities to use. This often leads to developers creating complex SSH aliases that require modifying the `git clone` URL, which is inconvenient.

## 2. The Solution: Directory-Based Context Switching ✅

This guide details a powerful, seamless method that allows you to copy and paste SSH URLs directly from any Git provider without modification. The correct SSH key and user identity (name/email) will be used automatically.

The magic lies in a combination of two features:

1.  **A Disciplined Folder Structure:** We organize repositories into dedicated parent folders based on the identity they belong to.
2.  **Git's Conditional Includes:** We configure Git to load different settings based on the directory a command is run from. This allows Git to automatically "switch contexts" for you.

---

## 3. Step-by-Step Configuration

### Step 1: Generate Dedicated SSH Keys 🔑

First, we need a unique SSH key for each identity. This is crucial for security and proper access management. We'll use the modern and secure `ed225519` algorithm.

Open your terminal and run the following commands. When prompted, it is highly recommended to add a secure and unique passphrase to each key.

**1. Personal GitHub Key:**
```shell
ssh-keygen -t ed25519 -C "personal-github-user@example.com" -f ~/.ssh/id_ed25519_github_personal
```

**2. Personal Bitbucket Key:**
```shell
ssh-keygen -t ed25519 -C "personal-bitbucket-user@example.com" -f ~/.ssh/id_ed25519_bitbucket_personal
```

**3. Organization/Work Key:**
```shell
ssh-keygen -t ed25519 -C "work-user@organization.com" -f ~/.ssh/id_ed25519_organization
```

You will now have three new key pairs in your `~/.ssh/` directory.

### Step 2: Add Public Keys to Your Git Accounts ☁️

The Git hosting services need to know about your public keys to grant you access.

For each key, display the **public** part (`.pub` file) in your terminal. Copy the entire output for each command.

**1. Get Personal GitHub Public Key:**
```shell
cat ~/.ssh/id_ed25519_github_personal.pub
```
* Go to **GitHub** -> Settings -> SSH and GPG keys -> New SSH key. Paste the key there.

**2. Get Personal Bitbucket Public Key:**
```shell
cat ~/.ssh/id_ed25519_bitbucket_personal.pub
```
* Go to **Bitbucket** -> Personal settings -> SSH keys. Paste the key there.

**3. Get Organization Public Key:**
```shell
cat ~/.ssh/id_ed25519_organization.pub
```
* Go to your **Organization's Bitbucket/GitHub** -> Personal settings -> SSH keys. Paste the key there.

### Step 3: Create the Foundational Directory Structure 📂

This method's success depends entirely on a consistent folder structure. Create a parent directory for your development work (e.g., `~/dev`), and then create subdirectories for each context.

```shell
# Create the main development folder
mkdir ~/dev
```

```shell
# Create folders for each context
mkdir -p ~/dev/organization
mkdir -p ~/dev/personal/github
mkdir -p ~/dev/personal/bitbucket
```

Your structure should look like this. All future repositories **must** be cloned into the appropriate folder.
```
/Users/youruser/
└── dev/
    ├── organization/
    │   └── (All your work repos will go here)
    └── personal/
        ├── github/
        │   └── (Your personal GitHub repos will go here)
        └── bitbucket/
            └── (Your personal Bitbucket repos will go here)
```

### Step 4: Simplify Your SSH Config ⚙️

Because Git will be handling the key selection, our SSH configuration in `~/.ssh/config` becomes very simple. Its only job is to provide basic connection parameters.

Open or create `~/.ssh/config` and add the following:

```ini
# General settings for Git hosts
Host github.com bitbucket.org
  User git
  IdentitiesOnly yes
  PasswordAuthentication no
```

* `User git`: Always use the `git` user for authentication.
* `IdentitiesOnly yes`: Prevents SSH from trying random keys.
* `PasswordAuthentication no`: A good security practice to ensure you're only using SSH keys.

### Step 5: Configure Git with Conditional Logic 🧠

This is the core of the setup. We will create three separate `.gitconfig` files that work together.

#### a) The Main Config (`~/.gitconfig`)

This file defines your **default** identity and sets up the rules for loading the other configs.
Open `~/.gitconfig` and add/edit it to look like this:

```shell
[user]
    # This is your DEFAULT identity.
    # It will be used for any repo NOT inside the specific folders.
    name = Your Personal GitHub Name
    email = personal-github-user@example.com

[core]
    # This is the DEFAULT SSH command. It uses your personal GitHub key.
    sshCommand = "ssh -i ~/.ssh/id_ed25519_github_personal"

### CONDITIONAL INCLUDES ###
# This section is the "brain" of the operation.

# If a repository is inside the '~/dev/organization/' directory...
[includeIf "gitdir:~/dev/organization/"]
    # ...then also load the settings from the following file.
    path = ~/.gitconfig-organization

# If a repository is inside the '~/dev/personal/bitbucket/' directory...
[includeIf "gitdir:~/dev/personal/bitbucket/"]
    # ...then also load the settings from this file.
    path = ~/.gitconfig-personal-bitbucket
```

#### b) The Organization-Specific Config

Create a new file named `~/.gitconfig-organization`. This file will **override** the defaults whenever you are working inside the `~/dev/organization/` folder.

```shell
[user]
    # Your work identity
    name = Your Work Name
    email = work-user@organization.com

[core]
    # IMPORTANT: Override the default sshCommand to use the organization key
    sshCommand = "ssh -i ~/.ssh/id_ed25519_organization"
```

#### c) The Personal Bitbucket Config

Create another new file named `~/.gitconfig-personal-bitbucket`. This overrides the defaults when you are inside `~/dev/personal/bitbucket/`.

```shell
[user]
    # Your personal Bitbucket identity
    name = Your Personal Bitbucket Name
    email = personal-bitbucket-user@example.com

[core]
    # Override the sshCommand to use your personal Bitbucket key
    sshCommand = "ssh -i ~/.ssh/id_ed25519_bitbucket_personal"
```

---

## 4. The New Workflow in Action ✨

Your setup is complete. You can now clone repositories by copying the SSH URL directly from the website, as long as you are in the correct directory first.

#### Scenario 1: Cloning an Organization Repository 🏢

1.  **Navigate** to the correct directory:
```shell
cd ~/dev/organization/
```

2.  **Go to** your organization's Bitbucket/GitHub and copy the SSH URL. It will look like `git@bitbucket.org:organization-name/project-name.git`.

3.  **Clone** it directly, without changes:
```shell
git clone git@bitbucket.org:organization-name/project-name.git
```

**How it works:** Git sees you are in `~/dev/organization/`, which matches the `includeIf` rule. It loads `~/.gitconfig-organization`, which overrides `core.sshCommand` to use the `id_ed25519_organization` key. Authentication succeeds, and any commits you make will use your work name and email.

#### Scenario 2: Cloning a Personal GitHub Repository 👤

1.  **Navigate** to the correct directory:
```shell
 cd ~/dev/personal/github/
```
2.  **Go to** GitHub and copy the SSH URL: `git@github.com:your-username/repo.git`.

3.  **Clone** it directly:
```shell
git clone git@github.com:your-username/repo.git
```

**How it works:** Git sees you are in `~/dev/personal/github/`. This path does **not** match any of the `includeIf` rules. Therefore, it uses the default settings from your main `~/.gitconfig` file, which correctly selects your personal GitHub key and identity.

## 5. Verifying Your Setup 🕵️‍♀️

To confirm which configuration is active in a given directory, you can ask Git to show you the origin of a specific setting.

Navigate into one of your project directories and run:
```shell
# Check which email address is currently active
git config --show-origin --get user.email
```

* When run inside `~/dev/organization/some-repo`, it will show the configuration comes from `~/.gitconfig-organization`.
* When run inside `~/dev/personal/github/some-repo`, it will show the configuration comes from `~/.gitconfig`.

You now have a robust, professional, and seamless Git environment. 👍