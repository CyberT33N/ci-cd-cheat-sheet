# ci-cd-cheat-sheet




# Git Tools

For efficient and guided creation of branches and commits in the terminal, please download the following tool:
- https://github.com/T-vK/git-tools/tree/master

  
<br><br>
<br><br>

## Git Tools for Guided Branch and Commit Creation

<br><br>


### Commit Convention

We follow the Angular Commit Convention. This convention ensures a consistent and traceable structure for our commit messages. More information can be found here: [Conventional Commits](https://www.conventionalcommits.org/).

Our `git-tools` already includes the implementation of this convention to ensure that all commits adhere to the standards.

### Branching Strategy and Deployment Process

[Branching Strategy Documentation](https://privadent.atlassian.net/wiki/spaces/SOFTWAREEN/pages/655374)

---

## Overview

This documentation describes the branching strategy and deployment process for our project. We use the `main`, `develop`, `Feature-Branches`, and `Feature-Dev-Branches`. This structure ensures that stable and tested code is deployed to production.

### Branches and Their Usage

#### 🛠 Hotfix Branches

Hotfix branches are similar to feature branches but are branched directly from the main branch. They are used for quick fixes of critical bugs and are immediately deployed to production.

**CI/CD:**
- Runs tests automatically
- Builds apps automatically
- Builds Docker images for hotfixes
- Tags Docker images with hotfix release versions
- Pushes Docker images to Nexus
- Pushes Docker images to Google Cloud
- Creates hotfix releases for each push
- Manual deployment task for Test Namespace and Production Namespace

**Rules:**
- Each hotfix branch must be formatted as `hotfix/{DESCRIPTION}`
- Full Regex: `/^hotfix/[\\w+-]*\\$/`
  
Example: `hotfix/critical-security-fix`

Once a hotfix is deployed and functioning:
- Changes must also be merged into the develop branch by creating a feature branch and submitting a pull request.

**Warning:** 
- If multiple hotfixes are deployed, the second hotfix branch must be branched from the first hotfix branch. Otherwise, deployment changes may be lost.

<br><br>
<br><br>

#### 🚀 Main Branch (Stable Releases)

The `main` branch is the production branch. All changes merged into the main branch should be deployed to production.

**Rules:**
- No direct pushes to this branch.
- Changes must come through pull requests from the develop or hotfix branch.
- No squashing; all commits from the develop branch must appear in the main branch.
- No additional commits (e.g., merge commits).

**Purpose:**  
The `main` branch represents the current state of production.

**Deployment:**  
After merging `develop` into `main`, the application is deployed to the staging environment. Once tests on staging are successful, the deployment proceeds to production.

**CI/CD:**
- Tags Docker images with commit hash on stable release version
- Pushes Docker images to Nexus
- Creates stable releases for each accepted merge request
- Manual deployment task for Test Namespace
- Manual or automated deployment task for Production Namespace

**Pre-merge Requirements:**
- All tests must be successfully executed (see below: Automated QA Checks and Manual QA Checks).


<br><br>
<br><br>

#### 🛠️ Develop Branch (Beta Releases)

The `develop` branch serves as the integration branch. All features that are ready to be merged but still need testing or validation are placed here.

**Rules:**
- No direct pushes to this branch.
- Changes must come through pull requests from feature branches.
- No squashing; all commits from the feature branch must appear in the develop branch.
- No additional commits (e.g., merge commits).

**Purpose:**  
The `develop` branch contains the latest stable developments that are not yet ready for production.

**Merge:**  
Feature branches are regularly merged into the develop branch. Once new features are implemented and tested, they are merged into `main`.

**CI/CD:**
- Unit tests: Test individual functions for correctness.
- Integration tests: Test the interaction between different modules.
- E2E tests: Simulate user interactions to ensure overall functionality.
- Static code analysis: Looks for bugs and security vulnerabilities.
- Tags Docker images with commit hash on beta release version.
- Pushes Docker images to Nexus
- Creates unstable/beta releases for each accepted merge request
- Manual deployment task for Test Namespace

**Manual QA Checks:**
- UI/UX Review: Ensures design consistency and usability.

<br><br>
<br><br>

#### 🌱 Feature Branches

Feature branches are used for developing new features or bug fixes. These branches are based on the `develop` branch.

**Rules:**
- Each feature branch must be formatted as `{TYPE}/{JIRA_TICKET_ID}/{DESCRIPTION}/main`.
- Types: feat|fix|ci|build|docs|style|refactor|perf|test
- Full Regex: `/^(feat|fix|ci|build|docs|style|refactor|perf|test)\\/[\\w+-]*\\/([\\w+-]*)\\/main$/`

Examples:
- `feat/CCS-1/new-feature-foo/main`
- `refactor/CCS-1/new-directory-structure/main`
- `fix/CCS-1/bugfix-ui-freezing/main`

Each feature branch represents a Jira ticket (and all of its sub-tasks).  
Each commit represents a Jira ticket (and all of its sub-tasks).

Each commit follows the Angular Commit Convention with the ticket ID as the scope:
```
feat(CCS-1112): Add new feature
```

**Merge:**  
Once a feature is fully developed, it is merged into `develop` via a pull request.

**CI/CD:**
- Unit tests: Ensure each function works correctly.
- Linting & code-style checks: Prevent styling issues and inconsistencies.
- Security scans: Check for vulnerabilities.
- Builds apps automatically.
- Builds Docker images for the apps.
- Tags Docker images with commit hash.
- Pushes Docker images to Nexus.
- Manual deployment task for Test Cluster.

<br><br>
<br><br>

#### 🔧 Feature-Dev Branches

Feature-dev branches are used for the internal, detailed development of a feature. They serve as "working branches" for developers, which are later used as the base for feature branches.

**Rules:**
- Each feature-dev branch must be formatted as `{TYPE}/{JIRA_TICKET_ID}/{DESCRIPTION}/{3_LETTER_DEV_NAME}`.
- Types: feat|fix|ci|build|docs|style|refactor|perf|test
- Full Regex: `/^(feat|fix|ci|build|docs|style|refactor|perf|test)\\/[\\w+-]*\\/([\\w+-]*)\\/\\w\\w\\w$/`

Examples:
- `feat/CCS-1/new-feature-foo/ofi`
- `refactor/CCS-1/new-directory-structure/dde`
- `fix/CCS-1/bugfix-ui-freezing/tvk`

Each feature-dev branch represents a Jira ticket (and all its sub-tasks).  
No commit convention required.

**Merge:**  
When development is complete, the feature-dev branch is squash merged into the corresponding feature branch.

---

## QA Checks and Testing

### 🛡️ Automated QA Checks

Automated QA checks ensure that code is stable and of high quality before reaching production.

- **Unit Tests:**  
  Test individual functions and classes for correctness.

- **Integration Tests:**  
  Check if different modules work together.

- **End-to-End (E2E) Tests:**  
  Simulate real user interactions to ensure the application functions as a whole.

- **Linting & Code-Style Checks:**  
  Ensure code consistency and readability.

- **Static Code Analysis:**  
  Finds potential bugs and security vulnerabilities.

- **Performance Checks:**  
  Test load times and resource usage.

- **Security Scans:**  
  Check for known vulnerabilities in dependencies.

### 🔍 Manual QA Checks

Manual QA checks are essential to ensure a high-quality user experience and to test the application for unexpected errors.

- **Feature Testing:**  
  Manually test new features to ensure they work as expected.

- **UI/UX Review:**  
  Ensure the application’s design is consistent and user-friendly.

- **Regression Testing:**  
  Ensure existing features work correctly after changes or additions.

- **Exploratory Testing:**  
  Test without predefined test cases to uncover unforeseen bugs and validate the application under various conditions.
