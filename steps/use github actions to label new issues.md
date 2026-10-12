# Use GitHub Actions to label new issues

The workflow [label-new-issue.yml](../.github/workflows/label-new-issue.yml) runs whenever an issue is **opened** and adds the `Triage/new` label using the `gh` CLI.

## Steps to trigger it

1. Make sure `label-new-issue.yml` is committed and pushed to the repository's default branch (issue-triggered workflows only run from the default branch).
2. Confirm Actions is enabled: **Settings > Actions > General > Allow all actions**.
3. Confirm the workflow token can write issues: **Settings > Actions > General > Workflow permissions** (the workflow already requests `issues: write` and `contents: read`).
4. Optionally create the label `Triage/new` under **Issues > Labels** (`gh issue edit --add-label` fails if the label doesn't exist).
5. Open a new issue: **Issues > New issue**, enter a title and body, then click **Submit new issue**.
   - Or via CLI: `gh issue create --title "Test issue" --body "Testing label workflow"`
6. Open the **Actions** tab and select the **Label New Issue** run that started.
7. Verify the run succeeded, then check the issue: it should now have the `Triage/new` label.

## Troubleshooting

- No run appears: the workflow isn't on the default branch, or Actions is disabled.
- Run fails with "label not found": create the `Triage/new` label (step 4).
- Run fails with a permissions error: check the workflow permissions (step 3).
- Editing or reopening an issue does not trigger it; only the `opened` event does.
