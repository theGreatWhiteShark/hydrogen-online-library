# Hydrogen Online Library

A Git-based artifact library for the
[Hydrogen](https://github.com/hydrogen-music/hydrogen) drum machine. This
repository is an example for a decentralized, versioned collection of Hydrogen
artifacts (drumkits, patterns, and songs) with automatic index generation via
CI/CD. Fork it and add your own artifacts to provide your own, personal online
library!

## Overview

This repository serves as a template library using which users can share their
own Hydrogen artifacts. When artifacts are added to the repository, a GitLab CI
pipeline automatically:

1. Scans all `.h2drumkit`, `.h2pattern`, and `.h2song` files (can be located
   both at top-level and in arbitrary subfolders)
2. Extracts metadata from each artifact
3. Generates a structured `index.json` file with permalinks
4. Deploys the index to a dedicated `library` branch

Users can then configure their Hydrogen application to consume artifacts from this library using the permalink to the `index.json` file.

## Supported Artifact Types

- **`.h2drumkit`** — Hydrogen drumkit archives (tar format containing drumkit.xml)
- **`.h2pattern`** — Hydrogen pattern files
- **`.h2song`** — Hydrogen song files

Supports formats as old as Hydrogen version 0.9.3.

## Adding Artifacts

### Prerequisites

1. **Fork and clone** this repository
2. **Add push permission for CI runner** using the settings of your forked repo
   Setttings > CI/CD > Job token permissions > Check "Allow Git push requests to the repository"

### Commit and Push

```bash
# Add your artifacts
git add drumkits/your-kit.h2drumkit
git add patterns/your-pattern.h2pattern
git add songs/your-song.h2song

# Commit with a descriptive message
git commit -m "Add acoustic jazz drumkit and basic patterns"

# Push to your fork
git push origin main
```

## CI/CD Pipeline

This repository supports both GitLab CI and GitHub Actions for automatic index generation. Choose the platform that matches your hosting platform.

### GitLab CI

The GitLab CI pipeline (`.gitlab-ci.yml`) automatically processes artifact additions:

#### Pipeline Stages

1. **Build** — Compiles the `hydrogen-index` tool from the submodule
2. **Index** — Scans artifacts and generates `index.json` with metadata and permalinks
3. **Deploy** — Pushes `index.json` to the `library` branch

#### Pipeline Triggers

The pipeline runs automatically on:
- Pushes to `main` or `master` branches
- Merge requests that are merged into `main` or `master`

#### GitLab Setup Requirements

1. **Enable CI/CD** in your repository settings
2. **Configure job token permissions**:
   - Go to Settings > CI/CD > Job token permissions
   - Check "Allow Git push requests to the repository"

### GitHub Actions

The GitHub Actions workflow (`.github/workflows/build-and-deploy-index.yml`) provides equivalent functionality:

#### Workflow Jobs

1. **build-indexer** — Compiles the `hydrogen-index` tool from the submodule
2. **generate-index** — Scans artifacts and generates `index.json` with metadata and permalinks
3. **deploy-index** — Pushes `index.json` to the `library` branch

#### Workflow Triggers

The workflow runs automatically on:
- Pushes to `main` or `master` branches
- Manual workflow dispatch (via GitHub Actions UI)

#### GitHub Setup Requirements

1. **Enable Actions** in your repository settings
2. **Grant write permissions**:
   - Go to Settings > Actions > General > Workflow permissions
   - Select "Read and write permissions"
   - Check "Allow GitHub Actions to create and approve pull requests"

### Pipeline Triggers (Both Platforms)

Both CI systems run automatically on:
- Pushes to `main` or `master` branches
- Merge requests/pull requests that are merged into `main` or `master`

### Platform Comparison

| Feature              | GitLab CI                                    | GitHub Actions                                 |
|----------------------|----------------------------------------------|------------------------------------------------|
| **Configuration**    | `.gitlab-ci.yml`                             | `.github/workflows/build-and-deploy-index.yml` |
| **Artifact Storage** | Built-in artifacts                           | Built-in artifacts                             |
| **Deployment Auth**  | CI_JOB_TOKEN                                 | GITHUB_TOKEN                                   |
| **Manual Trigger**   | Via pipeline UI                              | Via workflow_dispatch                          |
| **Permissions**      | Job token scope settings                     | Workflow permissions                           |
| **Permalink Format** | `gitlab.com/namespace/repo/-/raw/branch/...` | `github.com/namespace/repo/raw/branch/...`     |
| **Setup Complexity** | Medium                                       | Low                                            |

**Choosing a Platform:**
- **GitLab CI**: Use if your repository is hosted on GitLab
- **GitHub Actions**: Use if your repository is hosted on GitHub
- Both provide identical functionality for this use case

### Generated Index

The `index.json` file contains:

- **Metadata** for each artifact (name, author, license, version, etc.)
- **SHA-256 hashes** for integrity verification
- **Permalinks** to each artifact in the repository (platform-specific URLs)
- **Self-hash** of the index file for validation

Example structure:

```json
{
  "version": "0.1.0",
  "created": "2026-05-25T09:57:55",
  "patternCount": 4,
  "songCount": 17,
  "drumkitCount": 3,
  "patterns": [...],
  "songs": [...],
  "drumkits": [
    {
      "type": "drumkit",
      "name": "TR808EmulationKit",
      "url": "https://github.com/namespace/repo/raw/library/drumkits/TR808EmulationKit.h2drumkit",
      "hash": "abc123...",
      "author": "Author Name",
      "license": "CC BY",
      "instruments": 12,
      "samples": 12
    }
  ],
  "hash": "self-hash-of-index-file"
}
```

**Note**: The `url` field format differs by platform:
- **GitLab**: `https://gitlab.com/namespace/repo/-/raw/library/path/to/artifact`
- **GitHub**: `https://github.com/namespace/repo/raw/library/path/to/artifact`

## Consuming the Library

### Getting the Index Permalink

After your artifacts are merged, the `index.json` is available at:

**GitLab:**
```
https://gitlab.com/<namespace>/<repository>/-/raw/library/index.json
```

**GitHub:**
```
https://github.com/<namespace>/<repository>/raw/library/index.json
```

Replace `<namespace>` and `<repository>` with your project's path.

### Configuring Hydrogen

In the `Hydrogen` application, add the library permalink:

1. Open an "Online Import" dialog
2. Hit the "Sources" button and select "Add Source"
3. Add a new library with the permalink to `index.json`
4. Hydrogen will now be able to browse and download artifacts from this library

### Example Permalinks

**GitLab Example:**
```
https://gitlab.com/theGreatWhiteShark/hydrogen-online-library/-/raw/library/index.json
```

**GitHub Example:**
```
https://github.com/theGreatWhiteShark/hydrogen-online-library/raw/library/index.json
```

## Local Development

### Building the Index Locally

To generate the index locally without running the full CI pipeline:

```bash
# Ensure the submodule is checked out
git submodule update --init --recursive

# Build the hydrogen-index tool
cd hydrogen-index
go build -o ../hydrogen-index .
cd ..

# Generate the index (GitLab permalinks)
./hydrogen-index scan \
  --provider gitlab \
  --repo theGreatWhiteShark/hydrogen-online-library \
  --branch library \
  --output index.json \
  --exclude hydrogen-index

# OR generate the index (GitHub permalinks)
./hydrogen-index scan \
  --provider github \
  --repo theGreatWhiteShark/hydrogen-online-library \
  --branch library \
  --output index.json \
  --exclude hydrogen-index

# Validate the generated index
./hydrogen-index validate index.json
```

### Testing with Different Providers

**For GitLab:**
```bash
./hydrogen-index scan \
  --provider gitlab \
  --repo namespace/repository \
  --branch library \
  --output index.json \
  --exclude hydrogen-index
```

**For GitHub:**
```bash
./hydrogen-index scan \
  --provider github \
  --repo namespace/repository \
  --branch library \
  --output index.json \
  --exclude hydrogen-index
```

**For relative paths (no permalinks):**
```bash
./hydrogen-index scan --output index.json --exclude hydrogen-index
```

## Troubleshooting

### Pipeline Failures

If the CI pipeline fails:

**GitLab:**
1. Check the pipeline logs in GitLab CI/CD
2. Verify artifact files are valid Hydrogen formats
3. Ensure the submodule is properly initialized
4. Check for XML parsing errors in artifact metadata
5. Verify job token permissions are enabled

**GitHub:**
1. Check the workflow logs in GitHub Actions
2. Verify artifact files are valid Hydrogen formats
3. Ensure the submodule is properly initialized
4. Check for XML parsing errors in artifact metadata
5. Verify workflow permissions are set to "Read and write"

### Index Not Updating

If the `library` branch doesn't update:

**GitLab:**
1. Verify the pipeline completed successfully
2. Check that changes were pushed to `main` or `master`
3. Ensure the `deploy-index` job has proper Git permissions
4. Review the deploy job logs for authentication errors
5. Check job token scope settings

**GitHub:**
1. Verify the workflow completed successfully
2. Check that changes were pushed to `main` or `master`
3. Ensure workflow permissions include write access
4. Review the deploy job logs for authentication errors
5. Check that Actions are enabled in repository settings

### Invalid Artifacts

If artifacts fail to parse:

1. Open the artifact in Hydrogen to verify it's valid
2. Check XML syntax in drumkit.xml files
3. Ensure all referenced samples exist in drumkit archives
4. Validate format version is supported (>=0.9.3)

## License

This repository is licensed under GPLv3, consistent with the Hydrogen project.

Individual artifacts may have different licenses as specified in their metadata.

## Links

- [Hydrogen Drum Machine](https://github.com/hydrogen-music/hydrogen)
- [hydrogen-index Tool](https://github.com/hydrogen-music/hydrogen-index)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)

## Support

For issues or questions:
- Open an issue in this repository or [hydrogen-index](https://github.com/hydrogen-music/hydrogen-index)
- Check the [Hydrogen forum](https://github.com/hydrogen-music/hydrogen/discussions)
- Review the hydrogen-index documentation
