---
title: "Inner Loop with Continuous Integration"
date: 2024-07-08T07:56:02+01:00
tags: [engineering, vscode, ci, "continuous integration", "clean code", "code quality", Inner-loop, github, "branching strategy", branching]
draft: true
---


In this article, you'll learn how local feedback and continuous integration reinforce one another from edit to protected-branch merge. That matters because durable engineering comes from understanding trade-offs, not merely reproducing a command or pattern.

I huge part of DevOps is experimenting.  If you've no way to experiment in your inner loop then you need to do this via your outer loop using pipelines.

Optimize for fast feedback.  This is where code scan tools such as Linters or Analyzers come into their own.

As feature branches are hidden from others, we still need to test our changes with the latest merged code found in our mainline branch.  We can't rely on our developers to always pull frequently or even pre push to the remote branch.  We can however include these operations declaratively in our CI pipelines.  Each CI scenario can how their own nuances but essentially you're merging from mainline, building your code changes and testing in accordance to your CI scenario.  For example, the PR may include CDC tests, where as your feature may purely include L0 or L1 tests (class or namespace centric, fast tests).  Tip: Fast, inexpensive tests run first.



# IDE feedback

_Inner-loop_

# Feature Branch

### 1. Set Up GitHub Actions Workflow
Create a workflow that runs your build and tests on pull requests.

```yaml
name: CI for Feature Requests

on:
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'

    - name: Pull from main
      run: |
        git fetch origin main
        git merge origin/main

    - name: Install dependencies
      run: npm install

    - name: Run build
      run: npm run build

    - name: Run tests
      run: npm test
```

### 2. Configure Branch Protection Rules
1. Go to your repository on GitHub.
2. Navigate to `Settings` > `Branches`.
3. Under "Branch protection rules", click "Add rule".
4. Specify `main` as the branch name pattern.
5. Check the "Require status checks to pass before merging" option.
6. Select the build and test workflows that must pass before merging.
7. Optionally, check "Include administrators" to enforce these rules for repository admins as well.
8. Click "Create" or "Save changes".

### 3. Ensure Pull Requests are Merged Only When Checks Pass
With the branch protection rules in place, GitHub will prevent merging pull requests into the main branch unless the specified status checks (build and tests) pass. This ensures that only successfully built and tested code is merged into the main branch.

---

# Pull Request or Merge Request

You can modify the workflow to pull from the `main` branch before building. Here’s the updated workflow:

```yaml
name: CI for Pull Requests

on:
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '14'

    - name: Pull from main
      run: |
        git fetch origin main
        git merge origin/main

    - name: Install dependencies
      run: npm install

    - name: Run build
      run: npm run build

    - name: Run tests
      run: npm test

  merge:
    runs-on: ubuntu-latest
    needs: build
    if: github.event.pull_request.head.repo.full_name == github.repository
    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Merge pull request
      run: |
        git config --global user.name 'github-actions'
        git config --global user.email 'github-actions@github.com'
        git checkout main
        git merge ${{ github.event.pull_request.head.sha }}
        git push origin main
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Changes Made:
- Added a step `Pull from main` before the build step to fetch the latest changes from the `main` branch and merge them into the current branch.

---

# Mainline Branch Merge

# Strategies

## Release Flow

Always merge from mainline into release, never from release branch (in the case of hot or bug fixes) into mainline. one of the benefits with this protocol is thattThis insures ppl pull the latest from mainline.  <= reword

## Closing thought

Continuous integration should not be the first place a developer learns whether a change works; it should independently confirm evidence the inner loop already made cheap to obtain.
