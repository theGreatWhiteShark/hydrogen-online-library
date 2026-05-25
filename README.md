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

**Fork and clone** this repository

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

The GitLab CI pipeline automatically processes artifact additions:

### Pipeline Stages

1. **Build** — Compiles the `hydrogen-index` tool from the submodule
2. **Index** — Scans artifacts and generates `index.json` with metadata and permalinks
3. **Deploy** — Pushes `index.json` to the `library` branch

### Pipeline Triggers

The pipeline runs automatically on:
- Pushes to `main` or `master` branches
- Merge requests that are merged into `main` or `master`

### Generated Index

The `index.json` file contains:

- **Metadata** for each artifact (name, author, license, version, etc.)
- **SHA-256 hashes** for integrity verification
- **Permalinks** to each artifact in the GitLab repository
- **Self-hash** of the index file for validation

Example structure:

```json
{
  "version": "1.0.0",
  "generatedAt": "2026-05-25T12:00:00Z",
  "patternCount": 5,
  "songCount": 2,
  "drumkitCount": 3,
  "artifacts": [
    {
      "type": "drumkit",
      "name": "TR808EmulationKit",
      "path": "drumkits/TR808EmulationKit.h2drumkit",
      "permalink": "https://gitlab.com/namespace/repo/-/raw/library/drumkits/TR808EmulationKit.h2drumkit",
      "sha256": "abc123...",
      "metadata": { ... }
    }
  ]
}
```

## Consuming the Library

### Getting the Index Permalink

After your artifacts are merged, the `index.json` is available at:

```
https://gitlab.com/<namespace>/<repository>/-/raw/library/index.json
```

Replace `<namespace>` and `<repository>` with your project's path.

### Configuring Hydrogen

In the `Hydrogen` application, add the library permalink:

1. Open an "Online Import" dialog
2. Hit the "Sources" button and select "Add Source"
3. Add a new library with the permalink to `index.json`
4. Hydrogen will now be able to browse and download artifacts from this library

### Example Permalink

For this repository, the index is available at:

```
https://gitlab.com/theGreatWhiteShark/hydrogen-online-library/-/raw/library/index.json
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

# Validate the generated index
./hydrogen-index validate index.json
```

## Troubleshooting

### Pipeline Failures

If the CI pipeline fails:

1. Check the pipeline logs in GitLab
2. Verify artifact files are valid Hydrogen formats
3. Ensure the submodule is properly initialized
4. Check for XML parsing errors in artifact metadata

### Index Not Updating

If the `library` branch doesn't update:

1. Verify the pipeline completed successfully
2. Check that changes were pushed to `main` or `master`
3. Ensure the `deploy-index` job has proper Git permissions
4. Review the deploy job logs for authentication errors

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
