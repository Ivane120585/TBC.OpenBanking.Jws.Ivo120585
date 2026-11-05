# Source and release provenance

## Source lineage

The earliest commit available in this repository's Git history is `d23e60be4f9f4b80ab6a66ffbc41f80614f32791`, dated 2021-11-08. The same root commit (with no parent) exists in the public [`TBCBank/TBC.OpenBanking.Jws`](https://github.com/TBCBank/TBC.OpenBanking.Jws) repository, whose README is also linked from that commit. The original source and license retain TBC Bank copyright notices. This is evidence that the checked-in codebase includes inherited work; it does not support a claim that the entire codebase originated with its current maintainer.

Subsequent commits can show when particular changes entered this Git history and which Git identity was recorded for them. Git author metadata can be set by committers and is not, by itself, independent proof of a person's identity or legal rights. Review the original and later commits, source notices, applicable license, and any separate creation or assignment records when assessing a specific contribution or ownership claim.

## Verify released packages

For a tag-triggered release, the workflow builds and tests the source, computes SHA-256 checksums for both NuGet packages, attests those package digests, publishes the packages to NuGet, and attaches the packages and checksum manifest to the matching GitHub Release.

Download a release's assets and check the package bytes against the checksum manifest:

Replace `<tag>` with the release tag before running:

```sh
gh release download <tag> --repo Ivane120585/TBC.OpenBanking.Jws.Ivo120585
sha256sum -c SHA256SUMS.txt
```

Replace `<version>` with the package version you downloaded, then verify each package's attestation and require the expected repository workflow as its signer:

```sh
gh attestation verify Ivane120585.TBC.OpenBanking.Jws.<version>.nupkg \
  --repo Ivane120585/TBC.OpenBanking.Jws.Ivo120585 \
  --signer-workflow Ivane120585/TBC.OpenBanking.Jws.Ivo120585/.github/workflows/nuget.yml

gh attestation verify Ivane120585.TBC.OpenBanking.Jws.Http.<version>.nupkg \
  --repo Ivane120585/TBC.OpenBanking.Jws.Ivo120585 \
  --signer-workflow Ivane120585/TBC.OpenBanking.Jws.Ivo120585/.github/workflows/nuget.yml
```

The attestation binds the package digest to the GitHub Actions workflow and source revision used for the build. A checksum without a verified attestation can be replaced along with the package. These checks do not prove legal ownership, the human identity of a contributor, a relationship with a bank, or the absence of earlier work outside the recorded Git history.

The repository ruleset is currently disabled. Repository administrators should enable suitable rules that prevent force-pushes and branch deletion and require changes to be reviewed if protection against rewriting the default branch is needed.
