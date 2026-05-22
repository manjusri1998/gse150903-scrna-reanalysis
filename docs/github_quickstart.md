# GitHub Quickstart for This Project

Use this file as a small map for publishing the project.

## 1. Check What Changed

```bash
git status
```

## 2. Save a Version Locally

```bash
git add .
git commit -m "Add first transcriptomics portfolio project"
```

If Git asks who you are, run:

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Then repeat the commit command.

## 3. Create a Repository on GitHub

On GitHub, create a new empty repository. A good name would be:

```text
gse150903-scrna-reproduction
```

Do not add a README on GitHub because this local project already has one.

## 4. Connect This Folder to GitHub

GitHub will show commands after you create the repository. They will look like:

```bash
git remote add origin https://github.com/YOUR-USERNAME/gse150903-scrna-reproduction.git
git branch -M main
git push -u origin main
```

Replace `YOUR-USERNAME` with your GitHub username.

## 5. Normal Workflow After This

When you make more changes:

```bash
git status
git add .
git commit -m "Describe what changed"
git push
```

## Suggested Next Improvements

- Add a short paragraph comparing the reproduced figure to the paper figure.
- Move exploratory notes into Markdown cells inside the notebook.
- Add a second notebook that downloads or prepares the dataset.
- Add a small `results/tables/` folder for marker-gene CSV files if you decide to
  share selected result tables.
