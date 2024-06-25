# Git-Notes
## Table of Contents

| Section                                                                                             | Description                                                                   |
|-----------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------|
| [Using `git pull --rebase` Instead of `git pull`](#using-git-pull---rebase-instead-of-git-pull)     | Explanation of why to use `git pull --rebase` over `git pull`.                |
| [Updating Committer Name and Email in Git Repository](#updating-committer-name-and-email-in-git-repository) | Guide to updating the committer name and email using `git filter-branch`.     |
| [Updating Specific Commit Author and Committer Information in Git Repository](#updating-specific-commit-author-and-committer-information-in-git-repository) | Guide to updating author and committer for a specific commit.                 |

---

# Using `git pull --rebase` Instead of `git pull`

This section explains why you should use `git pull --rebase` instead of the regular `git pull` command and how it helps maintain a cleaner project history.

## Why Use `git pull --rebase`?

Assume John and Roney are working on the same project. John makes changes (b) on his local machine, and Roney makes changes (c) on his local machine. Roney pushes his changes to the remote branch. When John tries to push his changes, he cannot because John and Roney have the same ancestor commit (a). John uses the `git pull` command to fetch Roney's changes to his local machine. However, this results in a merge commit (d) because both John's and Roney's changes have the same ancestor commit (a). 

Over time, this can clutter the repository history with many unnecessary merge commits, making it difficult to read.

## Using `git pull --rebase`

When John hasn't pushed his changes yet and uses the `git pull --rebase` command, the remote changes are applied first, and then John's changes are added on top. This results in a cleaner history:
(a) -> (b) Remote -> (c) John's changes

### Command to Use

```sh
git pull --rebase
```
## Handling Conflicts
If conflicts occur while using git pull --rebase, you can abort the rebase process using:
```sh
git rebase --abort
```

Alternatively, you can use advanced rebase commands to resolve conflicts.
Or: For this situtation use `git pull`

---

# Updating Committer Name and Email in Git Repository

This guide explains how to update the committer name and email address in a Git repository using the `git filter-branch` command. This can be useful if you've changed your email address or if the incorrect information was used in previous commits.

## Steps to Update Committer Name and Email

1. **Open your terminal.**

2. **Navigate to your Git repository:**
    ```sh
    cd /path/to/your/repo
    ```

3. **Run the following command to update the committer name and email:**

    ```sh
    git filter-branch --env-filter '
    OLD_EMAIL="your-old-email@example.com"
    CORRECT_NAME="Your Correct Name"
    CORRECT_EMAIL="your-correct-email@example.com"
    if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
    then
        export GIT_COMMITTER_NAME="$CORRECT_NAME"
        export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
    fi
    if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
    then
        export GIT_AUTHOR_NAME="$CORRECT_NAME"
        export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
    fi
    ' --tag-name-filter cat -- --branches --tags
    ```

    Replace the following placeholders with your actual information:
    - `your-old-email@example.com`: Your old email address.
    - `Your Correct Name`: Your correct name.
    - `your-correct-email@example.com`: Your correct email address.

4. **Verify that the changes have been applied:**

    Check the history of your repository to ensure that the committer name and email have been updated correctly.

    ```sh
    git log
    ```

5. **Force-push the updated history to your remote repository:**

    If you are sure that the changes are correct, you need to force-push the updated history to your remote repository. Be very careful with this step as it will overwrite the history on the remote repository.

    ```sh
    git push --force --tags origin 'refs/heads/*'
    ```

## Important Notes

- **Backup Your Repository:** Before running the `git filter-branch` command, make sure to back up your repository. This process rewrites the commit history, and a backup ensures you can restore it if something goes wrong.
- **Force Push with Caution:** The `--force` option in `git push` is powerful and can potentially disrupt the work of others who share the repository. Make sure to communicate with your team before doing this.
- **Consider Using `git filter-repo`:** The `git filter-branch` command is powerful but also complex and potentially risky. For large repositories or complex history rewrites, consider using `git filter-repo` as a safer and faster alternative.

## Further Reading

- [Git Documentation: git filter-branch](https://git-scm.com/docs/git-filter-branch)
- [GitHub Help: Changing author info](https://help.github.com/en/github/using-git/changing-author-info)

By following these steps, you can ensure that your commits reflect the correct committer name and email address.

---

# Updating Specific Commit Author and Committer Information in Git Repository

This guide explains how to update the author and committer name and email address for a specific commit in a Git repository.

## Method 1: Using Interactive Rebase

1. **Find the commit hash:**
    ```sh
    git log
    ```

2. **Start an interactive rebase:**
    ```sh
    git rebase -i <commit-hash>^
    ```

3. **Mark the commit for editing:**
    In the interactive rebase editor, replace `pick` with `edit` for the commit you want to change.

4. **Change the commit author and committer:**
    ```sh
    git commit --amend --author="New Author Name <new-email@example.com>"
    ```

5. **Continue the rebase:**
    ```sh
    git rebase --continue
    ```

6. **Force push the changes:**
    ```sh
    git push --force
    ```

## Method 2: Using `git filter-repo`

1. **Install `git filter-repo`:** Follow the [official instructions](https://github.com/newren/git-filter-repo).

2. **Create a filter-repo script:**
    Save the following script as `change-author.sh`:
    ```sh
    #!/bin/sh

    COMMIT_HASH="commit-hash"
    NEW_NAME="New Author Name"
    NEW_EMAIL="new-email@example.com"

    git filter-repo --commit-callback '
    if commit.hash == b"'"$COMMIT_HASH"'":
        commit.author_name = b"'"$NEW_NAME"'"
        commit.author_email = b"'"$NEW_EMAIL"'"
        commit.committer_name = b"'"$NEW_NAME"'"
        commit.committer_email = b"'"$NEW_EMAIL"'"
    '
    ```

3. **Make the script executable:**
    ```sh
    chmod +x change-author.sh
    ```

4. **Run the script:**
    ```sh
    ./change-author.sh
    ```

5. **Force push the changes:**
    ```sh
    git push --force
    ```

## Important Notes

- **Backup Your Repository:** Before running these commands, make sure to back up your repository.
- **Force Push with Caution:** The `--force` option in `git push` is powerful and can potentially disrupt the work of others who share the repository. Communicate with your team before doing this.
- **Further Reading:**
    - [Git Documentation: Interactive Rebase](https://git-scm.com/docs/git-rebase)
    - [GitHub: git filter-repo](https://github.com/newren/git-filter-repo)

By following these steps, you can ensure that specific commits reflect the correct author and committer information.

