# Publish

This workflow publishes a Node.js package to the npm registry.  
It is designed to be reused across repositories via `workflow_call`.

[workflow definition](../.github/workflows/publish.yml)

## Purpose

The workflow provides a standardized and secure way to publish packages to npm, including:

- Node.js setup
- Optional version alignment with a GitHub release tag
- Support for npm 2FA using a one-time password (OTP)

## Trigger

This workflow is triggered using `workflow_call` from another repository.

## Required secrets

| Secret name     | Description                          | Required |
|-----------------|--------------------------------------|----------|
| `NPM_PUBLISH`   | npm token with publish permissions   | Yes      |

## Permissions

The workflow requires the following permissions:

- `contents: read` – to read repository contents
- `id-token: write` – to support secure authentication flows

## npm token requirements

For security reasons, the npm token used by this workflow must follow these rules:

- Use a **granular npm token** scoped only to the package(s) being published
- The token must have **publish-only permissions**
- The token should be **added shortly before each publish**
- The token must be **revoked immediately after the deployment completes**

## Environment protection

This workflow is executed in the `publish` environment.

Using a dedicated environment allows:

- Restricting access to sensitive secrets
- Enforcing manual approvals before publishing
- Applying environment-specific security rules

Secrets required for publishing must be scoped to this environment.

## Environment secret setup

To configure the `publish` environment and its secrets in GitHub:

1. Go to the repository **Settings**
2. Navigate to **Environments**
3. Click **New environment**
4. Create an environment named `publish`
5. Under **Environment secrets**, add:
   - `NPM_PUBLISH` with the npm publish token
6. Configure **Deployment branch restrictions**
7. (Optional) Configure:
   - Required reviewers
   - Wait timers

Only workflows explicitly targeting `environment: publish` will be able to access these secrets.

## Behavior

1. **Checkout repository**
   - Retrieves the package source code.

2. **Setup Node.js**
   - Uses Node.js version `24`
   - Configures the npm registry to `https://registry.npmjs.org`

3. **Version synchronization (optional)**
   - If triggered by a GitHub release creation:
     - Extracts the version from the release tag (expects `vX.Y.Z`)
     - Updates `package.json` if the version differs

4. **Wait for npm 2FA**
   - Pauses execution until a one-time password (OTP) is provided
   - Required when npm 2FA is enabled for publishing

5. **Publish**
   - Publishes the package to npm using:
     - The provided npm token
     - The supplied OTP
     - Public access

## Example usage

```yaml
name: Publish package to npm
on:
  release:
    types: [created]

permissions:
  id-token: write
  contents: read

jobs:
  publish:
    uses: expressjs/ci-workflows/.github/workflows/release.yml@main
    secrets:
      NPM_PUBLISH: ${{ secrets.NPM_PUBLISH }}
```

## Security overview

## npm token configuration and lifecycle management

Securing the npm publishing process is a critical aspect of modernizing Express’ supply-chain security. Publishing credentials represent a high-value attack surface, particularly when used in automated CI/CD environments.

To mitigate this risk, npm tokens used for publication must follow a strict lifecycle and configuration model. Granular tokens are scoped to a single package, enforce mandatory two-factor authentication, and are issued only for the shortest valid duration required to complete a release. Tokens are added immediately before publication and revoked as soon as the release process completes, ensuring no long-lived credentials persist beyond their intended use.

This approach significantly reduces the blast radius of potential CI compromises and aligns Express’ release process with modern supply-chain security best practices, while maintaining a practical and auditable workflow for maintainers.
