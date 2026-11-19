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

1. Open the [instructor repository](https://github.com/RafikHamza/gitleaks-secret-scanner). Select **Fork**, choose your account, and select **Create fork**.
2. Open `config.py` in your fork. It should read the value from the environment.

   ```python
   import os

   AWS_ACCESS_KEY_ID = os.environ.get("AWS_ACCESS_KEY_ID")
   ```

3. Open **Actions** and enable workflows if prompted. Select **Gitleaks Secret Scan**, then **Run workflow**. Choose `main`, leave the target set to `own-repository`, and start the run.
4. Open the completed run. Under **scan**, expand **Scan the selected repository with Gitleaks**. Confirm the step ran and the log says `no leaks found`. Save the run link.

Make all changes in your own fork using the pencil icon in GitHub. Commit to `main` for Steps 1–4. Each commit starts a new scan of your own repository. Match the run to the commit you just made before recording its result.

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

## 5. Scan another GitHub project

Your Gitleaks fork is the scanner. The instructor’s [StegoGuard practice branch](https://github.com/RafikHamza/stegoguard-image-forensics/tree/secret-scan-practice) is the target. It contains synthetic service settings in two fixed commits. You will review the finding and the proposed fix from the scanner repository.

1. In your Gitleaks fork, open **Actions** → **Gitleaks Secret Scan** → **Run workflow**. Keep `main` as the branch and select `stegoguard-before-fix` as the target.
2. Open **Scan the selected repository with Gitleaks** in the completed run. The scan should fail. Locate the rule ID, file, and line number. Follow the reported path in the [before-fix snapshot](https://github.com/RafikHamza/stegoguard-image-forensics/tree/59db83d6d5ae525d1f96eeea7b4ae894a0cbfd02) and explain what change you would request. Identify one safe configuration practice in the other practice files.
3. Run the workflow again with `stegoguard-after-fix`. Confirm the scanner reports `no leaks found`. Open the [fix commit](https://github.com/RafikHamza/stegoguard-image-forensics/commit/469eae77a53d8f8131578f0707de8f84f5e403ff) and explain how the change addresses the finding.
4. Compare the two logs. Explain why the new snapshot passes and why the earlier commit still contains the synthetic value.

The workflow checks the tracked files at the selected commit. It reads StegoGuard’s source files without running the target project. Use only the provided classroom snapshots for this exercise.

## Show me your work

When you finish, call me over and walk me through your project. Show me the Gitleaks runs from before and after you added your custom rule, your rule-test table, and the finding and fix from the separate StegoGuard target. Briefly explain why the result changed even though `config.py` stayed the same, why the AWS example is still visible in the earlier commit, and what you would do first if a real credential had been exposed.
