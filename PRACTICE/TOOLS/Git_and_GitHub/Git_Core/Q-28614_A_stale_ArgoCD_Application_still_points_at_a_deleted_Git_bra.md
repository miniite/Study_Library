---
type: "practice"
difficulty: "Intermediate"
---

# A stale ArgoCD Application still points at a deleted Git branch and shows Unknown, blocking your dashboard's health view. How would you audit and garbage-collect Applications whose source refs no longer exist, and automate that hygiene?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Recovering Lost Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Recovering_Lost_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

When ArgoCD applications point to deleted Git branches, it leads to stale references in the dashboard, making it difficult to monitor and manage your cluster's state. To address this issue, we need to audit and garbage-collect these Applications whose source refs no longer exist. This process involves identifying the problematic applications, removing them from the deployment state, and ensuring they are cleaned up automatically.

Triaging the Problem:

The question is primarily about managing stale ArgoCD references by automating the cleanup of applications that reference non-existent Git branches. The core challenge here is to identify these Applications and remove their references to ensure the health dashboard remains accurate and useful.

#### Assumptions
You have an existing ArgoCD deployment with multiple applications.
Your Git repository has been updated, deleting some branches.
These deleted branches are still referenced by some ArgoCD Applications.
You want a systematic approach to clean up these stale references.

#### First-Principles Solution
Step 1: Identify Stale Applications

To identify the Applications that reference non-existent Git branches, we can use the argo application list command or leverage the API to query the current state of applications and compare them against the available branches in the Git repository.

BASH
# List all ArgoCD applications
applications=$(argo application list --output json)

# Extract application names and source ref information
app_list=($(echo "$applications" | jq -r '.items[].metadata.name, .items[].spec.source.revision' | tr '\n' ' '))
Step 2: Check Git Repository for Existing Branches

Next, we need to query the Git repository to get a list of existing branches.

BASH
# Get all branch names from the Git repository (assuming it's hosted on GitHub)
git_branches=$(curl -s https://api.github.com/repos/your-repo-name/branches | jq -r '.[].name')
Step 3: Compare and Identify Stale Applications

Now, we compare the application source refs with the existing branches to identify which applications are referencing deleted branches.

BASH
stale_apps=()
for app in "${app_list[@]}"; do
    IFS=' ' read -r -a pair <<< "$app"
    app_name=${pair[0]}
    branch_ref=${pair[1]}

    if [[ ! " ${git_branches} " =~ " ${branch_ref} " ]]; then
        stale_apps+=("$app_name")
    fi
done
Step 4: Remove Stale Applications from ArgoCD

Once we have identified the stale applications, we can use argo application delete to remove them.

BASH
for app in "${stale_apps[@]}"; do
    argo application delete "$app"
done
Automating the Process:

To ensure this cleanup is automated and regular, you can create a scheduled job using tools like Kubernetes CronJobs or a cron job on your local system. For example, you could set up a cron job that runs every night to check for stale Applications.

BASH
# Example crontab entry
0 2 * * * /path/to/your/script.sh

#### Closing Summary

By following the steps above, we can systematically identify and clean up ArgoCD applications referencing deleted Git branches. Automating this process ensures that your dashboard's health view remains accurate and useful.

#### Common Mistakes/Follow-Ups
Incorrect Branch Names: Ensure you are comparing branch names correctly.
API Rate Limits: Be mindful of API rate limits when querying large repositories.
Permissions: Verify that the user running the script has sufficient permissions to delete applications.
Backup: Always consider backing up your application state before making changes.
Code Blocks:
BASH
applications=$(argo application list --output json)
app_list=($(echo "$applications" | jq -r '.items[].metadata.name, .items[].spec.source.revision' | tr '\n' ' '))
git_branches=$(curl -s https://api.github.com/repos/your-repo-name/branches | jq -r '.[].name')
stale_apps=()
for app in "${app_list[@]}"; do
    IFS=' ' read -r -a pair <<< "$app"
    app_name=${pair[0]}
    branch_ref=${pair[1]}

    if [[ ! " ${git_branches} " =~ " ${branch_ref} " ]]; then
        stale_apps+=("$app_name")
    fi
done
for app in "${stale_apps[@]}"; do
    argo application delete "$app"
done

