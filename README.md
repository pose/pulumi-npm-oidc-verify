# Pulumi npm OIDC publish verification

Private smoke-test repository for verifying Pulumi provider npm publishing with npm trusted publishing/OIDC.

This repository is intended to publish a disposable npm package using a `pulumi` binary built from a selected `pulumi/pulumi` ref. The workflow is `.github/workflows/verify.yml`; configure the npm package trusted publisher to this exact repository and workflow file.

## Before running

1. Pick an npm package name you control, for example `@pose/pulumi-npm-oidc-verify`.
2. If the package has never been published, bootstrap the first version with normal npm token auth outside this workflow. npm trusted publishing cannot create the first package release.
3. In npm, configure trusted publishing for:
   - GitHub owner: `pose`
   - Repository: `pulumi-npm-oidc-verify`
   - Workflow file: `verify.yml`
   - Package: the package name from step 1
4. Ensure the Pulumi branch/ref containing the `publishToNPM` OIDC change is available to GitHub Actions.

## Run

Use **Actions → Verify Pulumi npm OIDC publish → Run workflow** and provide:

- `pulumi_ref`: branch, tag, or SHA to build from `pulumi/pulumi`.
- `package_name`: npm package name configured for trusted publishing.
- `package_version`: a new, unpublished version.

The workflow intentionally does **not** set `NODE_AUTH_TOKEN`. It grants `id-token: write`, installs npm `11.19.0`, keeps `setup-node` `registry-url`, builds `bin/pulumi`, and runs:

```sh
pulumi package publish-sdk nodejs --path sdk/nodejs
```

Expected result: npm publish succeeds via OIDC provenance. If `pulumi package publish-sdk` still runs `npm whoami`, the job should fail with npm auth/401 before publishing.
