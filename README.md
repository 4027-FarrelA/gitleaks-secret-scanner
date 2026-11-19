# Gitleaks Secret Scanner with GitHub Actions

Find a hardcoded credential pattern, fix the source code, and test a detection rule of your own. Then use your scanner to review the before-and-after versions of a separate Python project. Complete the lab in your web browser.

## Why this project matters

Imagine a teammate opens a pull request for a new feature and accidentally commits an API key in a configuration file. The feature can work and its tests can pass while the key is exposed to anyone with access to the repository. Someone who obtains a real key could access data, make requests under the team’s account, or incur charges, depending on what the key allows. Deleting the line in a later commit can leave the original value in Git history.

Running Gitleaks in GitHub Actions gives the team a repeatable check during code review. In this lab, you will use the scan log to locate a finding, verify the fix, and test a rule with examples that should and should not match. Scanning the separate `tests` project lets you practice reviewing code you did not write. By the end, you should be able to explain what the scan checked, what a passing result tells you, and how you would respond if the credential were real.

| Repository | What you do there |
| --- | --- |
| Your fork of [gitleaks-secret-scanner](https://github.com/RafikHamza/gitleaks-secret-scanner) | Run every scan and make all your edits here. |
| [RafikHamza/tests](https://github.com/RafikHamza/tests) | Read the target code and review the fix in Step 5. |

**Use classroom examples only.** Never commit real passwords, API keys, access keys, tokens, private keys, or real `.env` files. You do not need an AWS account.

**AI is allowed and encouraged.** Use it to help interpret a log, draft a rule, or suggest test cases. Check its suggestions against your results. In the live walkthrough, explain what GitHub Actions runs, what Gitleaks checks, and how you verified your conclusions. A line-by-line explanation of the scanner or workflow code is optional.

## 1. Set up your fork and run a clean baseline

1. To create your fork, open the [instructor repository](https://github.com/RafikHamza/gitleaks-secret-scanner), click **Fork**, choose your own account, and click **Create fork**.
2. If you already have a fork, open its **Code** tab and select **Sync fork** → **Update branch**. If GitHub reports a conflict, ask me before continuing.
3. In your fork, click **Actions** and enable workflows if prompted. Select **Gitleaks Secret Scan** in the sidebar.
4. Click **Run workflow**, choose `main`, and check that the target dropdown offers `own-repository`, `tests-before-fix`, and `tests-after-fix`. If it still lists `stegoguard-*`, return to the **Code** tab and sync your fork.
5. Select `own-repository` and click the green **Run workflow** button.
6. Open the completed run, click the **scan** job, and expand **Scan the selected repository with Gitleaks**. The scanner step should pass and print `no leaks found`. Keep the run open or save its link for your walkthrough.

### How to edit a file and check its run

Open **Code** in your fork, select the file, and click the pencil icon. After editing, click **Commit changes**, commit directly to `main`, and return to **Actions**. GitHub starts an `own-repository` scan on each commit. Open the run for that commit and check the **scan** job. You can match runs to edits using the commit message or short SHA.

## 2. Add a classroom credential and find it in the log

1. In your fork, replace the contents of `config.py` with the following [published AWS example](https://docs.aws.amazon.com/IAM/latest/APIReference/API_CreateAccessKey.html#API_CreateAccessKey_Examples) and commit.

   ```python
   # Published AWS example for classroom testing.
   AWS_ACCESS_KEY_ID = "AKIAIOSFODNN7EXAMPLE"
   ```

2. Open the scan for that commit. It should fail. In the scanner log, find the rule ID `classroom-aws-example`, the file `config.py`, and the line number. Save the run link.

This course rule recognizes the published AWS example in `config.py`. A rule match identifies a pattern in the source. Credential validity requires separate evidence.

## 3. Fix the credential in your fork

1. Replace the contents of `config.py` with the following and commit.

   ```python
   import os

   AWS_ACCESS_KEY_ID = os.environ.get("AWS_ACCESS_KEY_ID")
   ```

2. Open the new run. It should pass with `no leaks found`. The credential now comes from the environment when the application runs.
3. Return to the failed run in Step 2 and follow its commit link. Verify that the old diff still shows the published example. Explain why a clean scan of the current commit leaves earlier commits unchanged.

## 4. Create and test your own detection rule

### First, test a worked example

1. Add `# LAB-DEMO-314159` on its own line at the end of `config.py` and commit. The scan should pass. Save the run link.
2. Open `.gitleaks.toml` and append the following block. Keep the existing rules and leave `config.py` as it is.

   ```toml
   [[rules]]
   id = "classroom-demo-code"
   description = "Classroom demo code"
   path = '''(^|/)config\.py$'''
   regex = '''\bLAB-DEMO-[0-9]{6}\b'''
   ```

3. Commit the TOML change. The scan should now fail and report `classroom-demo-code` in `config.py`. Save this run link as well. Compare the two runs to see what changed.

### Now design a rule for a fictional service

1. In the rule you just added, give it a new ID, replace the `LAB-DEMO-` prefix with a different one, and choose an exact number of digits. Keep the path restriction and commit.
2. In `config.py`, remove the worked-example comment. Add three different values that should match your rule and three near misses that should pass. Put each value on its own Python comment line. Use a wrong prefix, too few digits, and too many digits for the near misses. Commit all six together.
3. Open the scan for that commit. It should report three findings for your rule. Use the reported line numbers to check which three values matched. Make a small table with each test value, the expected finding, and the observed result.
4. Remove all six comments from `config.py`, keep your rule in `.gitleaks.toml`, and commit. Confirm the scanner runs and passes before moving to Step 5.

## 5. Scan the separate `tests` repository

### Scan the version with the synthetic leak

1. In **your Gitleaks fork**, select **Actions** → **Gitleaks Secret Scan** → **Run workflow**. Choose the `main` branch and select `tests-before-fix`. Start the run.
2. Open the completed run → **scan** → **Scan the selected repository with Gitleaks**. Find the `Snapshot:` line and match the SHA to the before-fix row below. The scan should fail.
3. In the log, find `generic-api-key` in `service_settings.py`, line 4. Gitleaks redacts the matching value in the log. Open the [before-fix source file](https://github.com/RafikHamza/tests/blob/a1bb4fccd5ba3a5c2ba07e76adbc9a8ae031a27b/service_settings.py#L4) to inspect its assignment.
4. Explain why Gitleaks found the value even though your custom rule targets `config.py`. `generic-api-key` is a built-in rule. Use the API-key assignment and the log’s entropy entry to explain why the synthetic value looked credential-like.
5. In the same snapshot, open [`.env.example`](https://github.com/RafikHamza/tests/blob/a1bb4fccd5ba3a5c2ba07e76adbc9a8ae031a27b/.env.example) and [`.gitignore`](https://github.com/RafikHamza/tests/blob/a1bb4fccd5ba3a5c2ba07e76adbc9a8ae031a27b/.gitignore). Find the empty credential placeholder and the rule that keeps a local `.env` file out of version control. Compare the README’s description of environment-based configuration with what line 4 actually does in this earlier snapshot.

### Scan the fixed version

1. From **your Gitleaks fork**, start another manual run on `main`, this time selecting `tests-after-fix`.
2. Open the scanner log, match its `Snapshot:` SHA to the after-fix row below, and verify that it reports `no leaks found`.
3. Open the [fix commit](https://github.com/RafikHamza/tests/commit/65706937738a63cbe9e9ea520c19ca30da069ab6). Identify the one line that changed and explain why reading `SERVICE_API_KEY` from the environment clears the finding. The earlier commit still contains the synthetic example.

| Target | Expected `Snapshot:` SHA | Expected scan result |
| --- | --- | --- |
| `tests-before-fix` | `a1bb4fccd5ba3a5c2ba07e76adbc9a8ae031a27b` | Failed with one built-in finding |
| `tests-after-fix` | `65706937738a63cbe9e9ea520c19ca30da069ab6` | Passed with `no leaks found` |

**If the scan errors before reporting findings**, check the log for a configuration error. The external scan uses your fork’s `.gitleaks.toml`. Check the syntax of the rule you appended in Step 4, correct it, and rerun the same target.

## Show me your work

When you finish, call me over and walk me through your project. Show me the before-and-after runs for your custom rule, your six-case test table, and the two `tests` scans. Explain why the built-in rule found the synthetic key, how you verified the fix, why the earlier commit still contains the value, and what you would do first if a real credential were exposed. [GitHub’s leaked-secret response guide](https://docs.github.com/en/code-security/tutorials/remediate-leaked-secrets/remediating-a-leaked-secret)
