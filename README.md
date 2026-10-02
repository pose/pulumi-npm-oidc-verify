# Pulumi npm trusted publishing demo

Private smoke-test/demo repository for showing Pulumi provider npm publishing with npm trusted publishing / GitHub Actions OIDC.

The workflow builds a `pulumi` CLI binary from a selected `pulumi/pulumi` ref, prepares a disposable Node.js SDK package, and publishes it with:

```sh
pulumi package publish-sdk nodejs --path ./sdk/nodejs
```

The publish step intentionally runs with `NODE_AUTH_TOKEN` unset. Authentication comes from GitHub Actions OIDC and npm trusted publishing.

## Demo package

Current package:

- npm package: `@pose/pulumi-npm-oidc-verify`
- trusted publisher repo: `pose/pulumi-npm-oidc-verify`
- workflow file: `.github/workflows/verify.yml`

The package must already exist on npm before trusted publishing can publish additional versions. Version `0.0.1` was bootstrapped/verified previously.

## What the workflow demonstrates

1. Builds Pulumi from the selected `pulumi_repo` + `pulumi_ref`.
2. Installs npm `11.19.0` for trusted publishing support.
3. Verifies the environment:
   - GitHub-hosted runner
   - `id-token: write` / OIDC request URL present
   - `NODE_AUTH_TOKEN` unset
   - npm `>= 11.5.1`
4. Runs `pulumi package publish-sdk nodejs --path ./sdk/nodejs`.
5. Shows that publishing succeeds without an npm token.
6. Polls npm metadata, but does not fail the demo if npm is still asynchronously validating the version.

## Running the demo

Use **Actions → Demo Pulumi npm trusted publishing → Run workflow**.

Recommended inputs:

- `pulumi_repo`: `pulumi/pulumi`
- `pulumi_ref`: `apose/pvd-4214-migrate-provider-npm-publishing-to-trusted-publishing-oidc`
- `package_name`: `@pose/pulumi-npm-oidc-verify`
- `package_version`: leave empty to auto-generate a unique version like `0.0.2-demo.<run_number>`

Or run from the CLI:

```sh
gh workflow run verify.yml \
  --repo pose/pulumi-npm-oidc-verify \
  -f pulumi_repo=pulumi/pulumi \
  -f pulumi_ref=apose/pvd-4214-migrate-provider-npm-publishing-to-trusted-publishing-oidc \
  -f package_name='@pose/pulumi-npm-oidc-verify'
```

After the publish step, the version may briefly show as **Validating** on npm before metadata and provenance are visible.

## Notes

- npm trusted publishing binds the npm package to the GitHub repository and workflow filename, not to the git ref.
- Do not set `NODE_AUTH_TOKEN` for the publish step; doing so exercises the token path rather than trusted publishing.
- Keeping `actions/setup-node` with `registry-url: https://registry.npmjs.org` is fine; no manual `.npmrc` workaround is required.
