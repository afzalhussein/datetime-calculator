# Combine subfolder repos into main project repo
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/37a4c5b2-a358-4136-ab41-d5c1f935b56b" />

It is common scenario: you have a main project folder that you initialized as a Git repository, and then inside it, you separately initialized `server` and `client` folders as *their own* Git repositories. This creates nested repositories, which GitHub (and Git itself) treats as independent entities, hence why you see three separate repositories.

To combine these into a single monorepository on GitHub, where `server` and `client` are just subdirectories within your main project's single repository, you'll need to strategically merge their histories.

The most robust and recommended way to do this, preserving the full commit history of your `server` and `client` sub-repositories within your main repository, is using **`git subtree`**.

Here's how you can accomplish this, assuming your current setup looks something like this:

```
my-main-project/
├── .git/  (main project repo)
├── server/
│   └── .git/ (server repo)
├── client/
│   └── .git/ (client repo)
└── ... other files
```

**Goal:** Transform it into a single repository on GitHub, like:

```
my-main-project/
├── .git/ (single combined repo)
├── server/
├── client/
└── ... other files
```

-----

### **Recommended Method: Using `git subtree` (Preserves History)**

This method integrates the history of your `server` and `client` repositories into your `my-main-project` repository.

**Prerequisites:**

1.  **Backup your work\!** Before doing anything, make a complete copy of your `my-main-project` folder. This is crucial in case anything goes wrong.
2.  **Ensure all changes are committed:** Make sure all pending changes in your `server`, `client`, and `my-main-project` repositories are committed.
3.  **Local work first:** Perform these steps locally.

**Steps:**

1.  **Navigate to your main project directory:**

    ```bash
    cd /path/to/my-main-project
    ```

2.  **Remove the nested `.git` folders:**
    These are the hidden Git repositories inside your `server` and `client` subfolders that are causing them to be separate repos.

    ```bash
    rm -rf server/.git
    rm -rf client/.git
    ```

    *Important: Do not remove the `.git` folder from `my-main-project`\!*

3.  **Add the `server` repository as a remote:**
    You'll need the URL of your `server` repository on GitHub.

    ```bash
    git remote add server-repo <URL_of_your_server_repo_on_GitHub>
    ```

    (e.g., `git remote add server-repo https://github.com/yourusername/my-server-repo.git`)

4.  **Integrate the `server` repository's history into a `server` subfolder:**
    This command pulls all the history from `server-repo` and places it into a `server/` directory within your main project.

    ```bash
    git subtree add --prefix server server-repo main --squash
    ```

      * `--prefix server`: Specifies the directory where the `server` content will live.
      * `server-repo`: The remote name you just added.
      * `main`: The branch name of the `server` repository you want to pull (could be `master` or `main`).
      * `--squash`: (Optional, but recommended for cleaner history) This squashes all the commits from the `server` repo into a single commit in your main repo's history, making the main repo's history less cluttered. If you want *all* individual commits from `server` to be preserved, omit `--squash`.

5.  **Repeat for the `client` repository:**

    ```bash
    git remote add client-repo <URL_of_your_client_repo_on_GitHub>
    git subtree add --prefix client client-repo main --squash
    ```

6.  **Clean up remotes (optional but good practice):**
    Once the history is merged, you no longer need these temporary remotes.

    ```bash
    git remote rm server-repo
    git remote rm client-repo
    ```

7.  **Remove the old repositories from GitHub (optional but recommended):**
    Once you've successfully merged the histories and confirmed everything is correct, you can delete the separate `server` and `client` repositories from GitHub. This is usually done through the GitHub website (Repository Settings -\> Danger Zone -\> Delete this repository).

8.  **Push your combined repository to GitHub:**
    Now, your `my-main-project` local repository contains the combined history. Push it to its corresponding GitHub repository.

    ```bash
    git push origin main # or 'master' depending on your main branch name
    ```

    If you haven't yet pushed `my-main-project` to GitHub, you'll need to set up its remote first:

    ```bash
    git remote add origin <URL_of_your_main_project_repo_on_GitHub>
    git branch -M main # if your default branch isn't already 'main'
    git push -u origin main
    ```

-----

### **Simpler Method: Manual Copy (Loss of Sub-Repo History)**

If the commit history of your `server` and `client` repositories *as separate entities* is not critical to preserve within the main repository's history (i.e., you just want their current files moved over), you can use a simpler approach.

1.  **Backup your work\!** (Always the first step).
2.  **Navigate to your main project directory:**
    ```bash
    cd /path/to/my-main-project
    ```
3.  **Ensure all changes are committed in all three repos.**
4.  **Remove the nested `.git` folders:**
    ```bash
    rm -rf server/.git
    rm -rf client/.git
    ```
5.  **Remove `.gitignore` from subfolders (optional but often necessary):**
    If the subfolders had their own `.gitignore` files, they might interfere with the main project's `.gitignore`. Decide if you want to integrate their rules into the main `.gitignore` or just delete them.
    ```bash
    rm server/.gitignore # Only if it exists and you don't need it
    rm client/.gitignore # Only if it exists and you don't need it
    ```
6.  **Add all files from the subfolders to the main repository's staging area:**
    ```bash
    git add .
    ```
7.  **Commit the changes:**
    ```bash
    git commit -m "Combined server and client projects into a single monorepo"
    ```
8.  **Push to GitHub:**
    ```bash
    git push origin main # or 'master'
    ```
    And delete the old `server` and `client` repositories from GitHub.

**Pros:** Much simpler steps.
**Cons:** The individual commit history of `server` and `client` from before the merge will not be part of the `my-main-project`'s history. It will appear as one large commit adding their current state.

-----

Choose the method that best suits your need for historical preservation. For most professional projects, `git subtree` is preferred for maintaining a clear and comprehensive commit history. Always test locally before pushing to your main remote repository\!
