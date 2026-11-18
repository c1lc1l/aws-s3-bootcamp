# Phase 5 - BONUS: automatic deployment with GitHub Actions (about 25 minutes)

> ⭐ **Optional.** Finish Phase 4 first. Your core workshop is complete without this phase.

**Goal:** every time you commit to `main`, GitHub automatically uploads `site/` to your S3 bucket.

By the end of this phase you can say:

- [ ] I stored AWS keys as **GitHub Secrets**, never in my code.
- [ ] I added the workflow and saw a green ✅ run on the **Actions** tab.
- [ ] I changed `index.html`, committed, and saw the change on my S3 website without using the S3 Console.

---

## Manual vs automated deployment

| Manual (Phase 4) | Automated - CI/CD (Phase 5) |
|---|---|
| You upload files in the S3 Console | A robot (GitHub Actions) uploads them for you |
| Easy to forget a file or upload to the wrong place | Same steps every time |
| Fine for one-off changes | How real teams ship many times a day |

**CI/CD** = *Continuous Integration / Continuous Delivery*: automatically building, testing and deploying every change.

## What you need

1. Your **CI/CD card** from a facilitator. Ask for it when you finish Phase 4. It holds an **access key ID** and a **secret access key**.
2. Your **bucket name** from Phase 4.
3. Your GitHub repository from Phase 2. Its `site/` folder must contain your **latest** website, because the workflow uploads exactly what is in GitHub.

> 🔒 Your CI/CD keys belong to a separate **deploy-only identity**. It cannot sign in to the Console, cannot create or change buckets,
> and can only upload/delete files in the bucket that is **registered to your username** (its `workshop-owner` tag). It is switched off after the workshop.

---

## Step 1 - Configure secrets and the bucket variable

In **your** repository: **Settings** (top tab) → left menu **Secrets and variables** → **Actions**.

**Secrets** tab → **New repository secret**, twice:

| Name (exactly) | Secret |
|---|---|
| `AWS_ACCESS_KEY_ID` | the access key ID from your CI/CD card (starts with `AKIA`) |
| `AWS_SECRET_ACCESS_KEY` | the secret access key from your CI/CD card |

**Variables** tab → **New repository variable**:

| Name (exactly) | Value |
|---|---|
| `S3_BUCKET` | the bucket name **you chose** in Phase 4, e.g. `my-first-cloud-portfolio` (no `s3://`, no `http://`) |

> Secrets are encrypted and hidden, even from you, after saving. Variables are visible.
> The bucket name isn't secret; the keys are.

## Step 2 - Add the workflow

1. Open `bonus/github-actions/deploy.yml` in your repository and copy all of its contents (use the **copy raw file** button).
2. Go back to the repository's main page → **Add file → Create new file**.
3. In the file name box type exactly:
   ```text
   .github/workflows/deploy.yml
   ```
   (Typing `/` creates folders. It starts with a **dot**.)
4. Paste the contents.

## Step 3 - Commit the workflow

**Commit changes…** → message `Add S3 deploy workflow` → **Commit directly to the main branch** → **Commit changes**.

This commit is a push to `main`, so **the workflow runs right away**.

## Step 4 - Watch it run

Click the **Actions** tab → open **Deploy website to S3** → click the run → **deploy**.
Each step shows ✅ or ❌. The last step prints your website address.

## Step 5 - Edit `index.html`

Open `site/index.html` → ✏️ → change something visible, for example your introduction text, then:

## Step 6 - Push (commit) to GitHub

**Commit changes** to `main`. In the browser, a commit to `main` *is* the push.
(Using Git Bash? Run `git add site/ && git commit -m "Update intro" && git push`.)

## Step 7 - Inspect GitHub Actions and refresh your website

1. **Actions** tab → a new run appears within seconds → wait for ✅ (about 30 seconds).
2. Open your S3 website endpoint → **Ctrl+F5** (hard refresh). Your change is live. 🎉

You can also rerun a deploy anytime: **Actions → Deploy website to S3 → Run workflow**.

---

## About `aws s3 sync` and `--delete`

The workflow runs:

```bash
aws s3 sync "site/" "s3://$S3_BUCKET" --no-progress
```

`sync` uploads only files that are **new or changed**. Files you deleted from `site/` **stay** in the bucket.
Adding `--delete` makes the bucket an exact mirror of `site/`. **Anything in the bucket that isn't in `site/` is deleted**, including images you uploaded by hand in the Console.
Only add `--delete` once *all* your files live in GitHub.

## When it fails ❌

Open the failed step and read the red message:

| Message contains | Cause | Fix |
|---|---|---|
| `Secret AWS_ACCESS_KEY_ID is missing` | Secret not created, or a typo in its name | Step 1; names must match exactly |
| `Variable S3_BUCKET is missing` | Created as a **secret** instead of a **variable** | Create it on the **Variables** tab |
| `is not a bucket name` | You entered `s3://…`, a URL, or a space | Enter only the bucket name |
| `InvalidClientTokenId` / `SignatureDoesNotMatch` | Key ID or secret pasted wrongly (extra space, swapped) | Delete and re-create both secrets carefully |
| `AccessDenied` on `ListObjectsV2` or `PutObject` | `S3_BUCKET` is not **your registered** bucket: a typo, a classmate's bucket, or a bucket that was never registered | Copy the exact name from the S3 Console; the bucket must be the one registered to your username |
| `NoSuchBucket` | Bucket name typo | Copy the name from the S3 Console |
| `site/index.html not found` | You moved or renamed the `site` folder | Website files must stay in `site/` |
| Workflow never runs | File is not at `.github/workflows/deploy.yml`, or you committed to another branch | Check the path (leading dot!) and branch `main` |
| Run is green but the site didn't change | Browser cache, or you edited a file outside `site/` | Ctrl+F5; edit `site/index.html` |

## Why keys and not something better?

Real projects use **OIDC** short-lived credentials: GitHub proves who it is to AWS, and no long-lived key is stored anywhere.
That requires the AWS account owner to register **each** repository in advance. We could not do that for 30 new repositories created today, so this
workshop uses keys that are **limited to your registered bucket**, **stored only in GitHub Secrets**, and **deactivated right after the event**.
They are completely separate from the Console password: changing or revoking one never affects the other.

**Never** commit keys to a repository: bots scan public GitHub for leaked keys within minutes.

⬅️ Back to [Phase 4](04-deploy-to-s3.md) · Showcase prep: copy your website URL and be ready to share one thing you learned.
