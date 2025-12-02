# Release-Please Prerelease and Release Example

This repository demonstrates how to implement a SOC 2 compliant release workflow using the [release-please-action](https://github.com/googleapis/release-please-action). The workflow automates version management, changelog generation, and GitHub release creation, ensuring a consistent and auditable release process.

## Overview

The release workflow consists of two main stages:
1. **Prerelease** - Creates release candidates (RC) for testing before final release
2. **Release** - Creates the final release after successful testing

Given all additional testing- and publish-workflows are implemented compliant, and the GitHub repository has properly configured branch protection rules. This template shows an approach which supports SOC 2 compliance by:
- Compatible with protected main branch rule (no direct commits to main)
- Maintaining a clear audit trail of all changes
- Enforcing a consistent release process
- Automating version management
- Providing traceability between code changes and releases
- Enforcing separation of duties between development and release processes
- Creating a controlled environment for testing through prereleases
- Providing evidence of testing before final release
- Generating comprehensive change management documentation
- Supporting access control through PR approval workflows
- Enabling risk assessment through the prerelease process

## Workflow Features

- **Automated Versioning**: Automatically increments version numbers based on [Conventional Commits](https://www.conventionalcommits.org/)
- **Changelog Generation**: Automatically creates and updates CHANGELOG.md with categorized changes
- **GitHub Releases**: Creates GitHub releases with appropriate tags
- **Version Synchronization**: Updates version information across multiple files
- **Prerelease Support**: Creates release candidates for testing before final release
- **Conditional Execution**: Runs different jobs based on whether it's a prerelease or final release

## How It Works

### Configuration Files

- `.github/prerelease-config.json`: Configuration for prerelease stage
- `.github/release-config.json`: Configuration for final release stage
- `.github/prerelease-manifest.json`: Tracks the prerelease version
- `.github/release-manifest.json`: Tracks the release version

### Workflow Stages

1. **Label Check**: Creates the necessary labels for release-please PRs
2. **Prerelease Preparation**: Creates or finalizes a prerelease pull request
3. **Prerelease Testing**: Runs tests before creating the prerelease
4. **Prerelease Creation**: Creates the prerelease and a subsequent release pull request
5. **Post-Prerelease Steps**: Runs any necessary steps after prerelease
6. **Release Creation**: Creates the final release
7. **Post-Release Steps**: Runs any necessary steps after release

### Version Management

In this example the workflow updates version information in:
- `version.txt`: Simple version tracking
- `.github/prerelease-manifest.json`: Version manifest for release-please
- `library/build.gradle.kts`: Version in build configuration (using special markers)

### Changelog Management

The CHANGELOG.md file is automatically updated based on the `changelog-sections` configuration in both the prerelease and release configuration files. The current configuration includes:

- Features (from `feat:` commits)
- Bug Fixes (from `fix:` commits)
- Code Refactoring (from `refactor:` commits)
- Miscellaneous Chores (from `chore:` commits)
- Build System and Dependencies (from `build:` commits)
- Documentation (from `docs:` commits)

Each section can be customized by modifying the `changelog-sections` array in the configuration files:

```json
"changelog-sections": [
  { "type": "feat", "hidden": false, "section": "Features" },
  { "type": "fix", "hidden": false, "section": "Bug Fixes" },
  { "type": "refactor", "hidden": false, "section": "Code Refactoring" },
  { "type": "chore", "hidden": false, "section": "Miscellaneous Chores" },
  { "type": "build", "hidden": false, "section": "Build System and Dependencies" },
  { "type": "docs", "hidden": false, "section": "Docs" }
]
```

You can add additional commit types, change section titles, or hide specific sections by setting `"hidden": true`. For more information on this, check the [Configuration File Reference](#Configuration-File-Reference) section.

## Setup

### Configure GitHub Environment (Required for Manual Approval)

Before using the workflow, you need to create a GitHub Environment:

1. Go to your repository **Settings**
2. Click on **Environments** in the left sidebar
3. Click **New environment**
4. Name it: `prerelease-approval`
5. Under **Deployment protection rules**, add:
   - ✅ **Required reviewers**: Add yourself or your team members
6. Click **Save protection rules**

This enables the manual approval button for prereleases (similar to GitLab's manual jobs).

## Usage

### Prerelease for Feature Branches (Label-Based)

To create a prerelease for any PR/feature branch:

1. Open your PR (e.g., `feature/my-feature`)
2. Add the label **`create-prerelease`** to the PR
3. The workflow automatically starts and creates a **prerelease PR** for that branch
4. Merge the prerelease PR to create a release candidate for testing

**Use case:** Test your feature branch changes with a real release candidate before merging to main!

### Manual Prerelease (With Approval for Main Branch)

When you push to main, the workflow runs automatically:

1. Push changes to main with [Conventional Commits](https://www.conventionalcommits.org/)
2. The workflow starts automatically
3. **Prerelease job pauses** and shows "Waiting for approval"
4. Click **"Review deployments"** button in the Actions tab
5. Click **"Approve and deploy"** to create the prerelease, or **"Reject"** to skip it
6. If approved, the workflow creates a **prerelease PR** with version like `1.10.7-rc.1`
7. When you merge that prerelease PR, it creates a release candidate for testing

**This is similar to GitLab's `when: manual` - the job appears in the pipeline but waits for you to click a button!**

### Automatic Release PR

When you push/merge to the main branch:

1. Push changes to main with [Conventional Commits](https://www.conventionalcommits.org/)
2. The workflow **automatically creates a release PR** with the next version (e.g., `1.10.7`)
3. Review and merge the release PR
4. The final release is created automatically

### Both PRs Can Coexist

You can have **both a prerelease PR and a release PR open at the same time**:
- **Prerelease PR** (e.g., `1.10.7-rc.1`) - Created manually for testing
- **Release PR** (e.g., `1.10.7`) - Created automatically when pushing to main

This allows you to test with release candidates while continuing to prepare the final release.

## Implementation Details

The workflow is implemented using GitHub Actions and the release-please-action. It requires:
- GitHub repository permissions for contents, pull-requests, and repository-projects
- Conventional commit messages for proper changelog generation and version bumping
- Configuration files for prerelease and release stages

## Configuration File Reference

This section provides details on how to extend and modify the configuration files used by release-please.

### Key Configuration Options Explained

- **release-type**: Specifies the release strategy. Options include:
  - `simple`: Basic generic versioning
  - `node`: For Node.js projects
  - `python`: For Python projects
  - `java`: For Java projects
  - `maven`: For Maven projects
  - `go`: For Go projects
  - `rust`: For Rust projects
  - And many others (see [release-please documentation](https://github.com/googleapis/release-please/blob/main/docs/customizing.md))

- **prerelease**: Boolean flag to indicate if this is a prerelease configuration

- **prerelease-type**: The type of prerelease to create (e.g., `rc`, `beta`, `alpha`)

- **versioning**: The versioning strategy to use:
  - `prerelease`: For prerelease versioning (adds suffixes like `-rc.1`)
  - `default`: Standard semantic versioning

- **changelog-sections**: Defines how different commit types are categorized in the changelog:
  - `type`: The commit type (from conventional commits)
  - `hidden`: Whether to hide this section in the changelog
  - `section`: The section title in the changelog

- **packages**: Defines the package structure for monorepos or single packages:
  - `.`: Represents the root package
  - `type`: The package type (e.g., `generic`, `node`, `java`)
  - `extra-files`: Additional files to update with version information

- **extra-files**: Files to update during version changes:
  - `type`: The file type (`generic`, `json`, `xml`, etc.)
  - `path`: Path to the file
  - `jsonpath`: For JSON files, the path to the version field
  - `marker`: For generic files, a marker pattern to identify version strings

### Customization Examples

#### Adding Custom Changelog Sections

```json
"changelog-sections": [
  { "type": "feat", "hidden": false, "section": "Features" },
  { "type": "fix", "hidden": false, "section": "Bug Fixes" },
  { "type": "perf", "hidden": false, "section": "Performance Improvements" },
  { "type": "refactor", "hidden": false, "section": "Code Refactoring" },
  { "type": "test", "hidden": false, "section": "Testing" },
  { "type": "chore", "hidden": true, "section": "Miscellaneous Chores" }
]
```

#### Updating Version in Multiple Files

```json
"extra-files": [
  {
    "type": "json",
    "path": "package.json",
    "jsonpath": "$.version"
  },
  {
    "type": "xml",
    "path": "pom.xml",
    "xpath": "//project/version"
  },
  {
    "type": "generic",
    "path": "src/version.py",
    "marker": "VERSION = \"${version}\""
  }
]
```

#### Configuring for a Monorepo

```json
"packages": {
  "packages/core": {
    "component": "core",
    "version-file": "version.txt"
  },
  "packages/ui": {
    "component": "ui",
    "version-file": "version.txt"
  }
}
```

### Best Practices

1. **Use Consistent Release Types**: Choose a release type that matches your project's language and structure.

2. **Define Clear Changelog Sections**: Customize changelog sections to match your team's commit conventions.

3. **Update All Version References**: Ensure all files containing version information are included in `extra-files`.

For more detailed information, refer to the official documentation:
- [release-please-action](https://github.com/googleapis/release-please-action)
- [release-please configuration](https://github.com/googleapis/release-please/blob/main/docs/customizing.md)
