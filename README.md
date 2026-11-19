# Gitleaks Secret Scanner with GitHub Actions

Find a hardcoded credential pattern, fix the source code, and test a detection rule of your own. You will complete the lab in your GitHub fork using the web browser.

## Using AI

You are encouraged to use AI to help interpret scan results, draft a detection rule, or suggest test cases. Check its suggestions against the workflow logs and explain your decisions in your own words. During the walkthrough, focus on how the app works at a high level. Explain what GitHub Actions runs, what Gitleaks checks, what a finding tells you, and how you verified the fix. A line-by-line explanation of the scanner or workflow code is optional.

## What you will learn

- Run a secret scan and identify the matching rule, file, and line number.
- Replace a hardcoded credential with a reference to an environment variable.
- Compare two scans of the same file to show what a new rule detects.
- Explain why a passing scan of the current commit leaves earlier commits unchanged.

**Use classroom examples only.** Never commit real passwords, API keys, AWS credentials, tokens, private keys, or real `.env` files. The AWS access key ID below appears in the [AWS documentation](https://docs.aws.amazon.com/IAM/latest/APIReference/API_CreateAccessKey.html#API_CreateAccessKey_Examples). You do not need an AWS account or any real credentials.

## 1. Fork the repository and run a clean baseline

1. Open the [instructor repository](https://github.com/RafikHamza/gitleaks-secret-scanner). Select **Fork**, choose your account, clear **Copy the main branch only** so you get the practice branch, and select **Create fork**.
2. Open `config.py` in your fork. It should read the value from the environment.

   ```python
   import os

   AWS_ACCESS_KEY_ID = os.environ.get("AWS_ACCESS_KEY_ID")
   ```

3. Open **Actions** and enable workflows if prompted. Select **Gitleaks Secret Scan**, then **Run workflow**. Choose `main` and start the run.
4. Open the completed run. Under **scan**, expand **Scan the current commit with Gitleaks**. Confirm the step ran and the log says `no leaks found`. Save the run link.

Make all changes in your own fork using the pencil icon in GitHub. Commit to `main` for Steps 1–4. In Step 5, make changes on `classroom-practice`. Each commit starts a new scan. Match the run to the commit you just made before recording its result.

## 2. Add the published AWS example and inspect the finding

1. Replace the contents of `config.py` with the following classroom example and commit.

   ```python
   # Published AWS example for classroom testing.
   AWS_ACCESS_KEY_ID = "AKIAIOSFODNN7EXAMPLE"
   ```

2. Open the workflow run for that commit. The scan should fail. In the scanner step, find the rule ID `classroom-aws-example`, the file `config.py`, and the matching line number. Save the run link and those three details.

Gitleaks checks for patterns. The lab rule explicitly recognizes this published example in `config.py`, and the workflow also uses Gitleaks’ built-in rules. A match tells you what pattern was found. It does not establish that a credential is active or that the scanner can detect every kind of secret.

## 3. Fix the source code and check the scope of the scan

1. Restore the original `import os` and `os.environ.get("AWS_ACCESS_KEY_ID")` code from Step 1. Commit and confirm the scanner reports `no leaks found`. Save this run link.
2. From the earlier failed run, follow the link to its commit. Confirm that the example is still visible in that commit’s diff.

The workflow scans the tracked files in the current commit. The fix removed the hardcoded example from the current version. For a real leak, revoke the exposed credential promptly, issue a replacement if needed, and update the systems that use it. [GitHub’s leaked-secret response guide](https://docs.github.com/en/code-security/tutorials/remediate-leaked-secrets/remediating-a-leaked-secret)

## 4. Create and test a custom rule

You will scan the same `config.py` before and after adding a classroom rule. The demo code below has no credentials behind it.

1. Append `# LAB-DEMO-314159` on its own line in `config.py`. Commit and confirm the scan passes. Save the run link.
2. Open `.gitleaks.toml` and append the following block at the end. Leave the existing rules and `config.py` as they are.

   ```toml
   [[rules]]
   id = "classroom-demo-code"
   description = "Classroom demo code"
   path = '''(^|/)config\.py$'''
   regex = '''\bLAB-DEMO-[0-9]{6}\b'''
   ```

3. Commit the rule. The next scan should fail with `classroom-demo-code` in `config.py`. Save the run link and the rule ID, file, and line number.
4. Delete the demo comment from `config.py` and commit. Confirm the final scan passes with the rule still enabled. Save the final run link.

Now adapt the example rule for a fictional service of your choice. Use a different prefix, pick an exact number of digits, and give the rule a new ID. Test three values that should match and three that should pass, including the wrong prefix, one digit too few, and one digit too many. Put one value at a time on its own comment line in `config.py`. Keep a small expected-versus-actual table for the walkthrough. Remove the last test value and confirm the workflow is green.

## 5. Review a realistic pull request

A teammate is adding settings for a fictional service. Your job is to decide whether the change is ready to merge.

1. In your fork, open **Pull requests** → **New pull request**. Set the base to `main` and compare it with `classroom-practice`. If the compare branch is missing, create `classroom-practice` from your fork’s `main` and use GitHub’s web editor to copy `service_settings.py`, `.env.example`, and `docs/service-setup.md` from the [instructor’s practice branch](https://github.com/RafikHamza/gitleaks-secret-scanner/tree/classroom-practice) into the same paths on your practice branch. Check that both branches belong to your fork, then create the pull request.
2. Open **Checks** on the pull request. In the Gitleaks scanner log, record the rule ID, file, and line number. Review the other changed files in **Files changed** and identify one safe configuration practice worth keeping. Decide whether to approve the change or request changes, and explain your decision.
3. Fix the finding on `classroom-practice` by reading the credential from the environment. Confirm that the new pull-request scan passes.
4. In **Commits**, find the earlier commit that introduced the sample. Be ready to explain what the new scan verified and what you would do first if the sample had been a real credential.

## Show me your work

When you finish, call me over and walk me through your project. Show me the Gitleaks runs from before and after you added your custom rule, your rule-test table, and the pull-request finding and fix. Briefly explain why the result changed even though `config.py` stayed the same, why the AWS example is still visible in the earlier commit, and what you would do first if a real credential had been exposed.
