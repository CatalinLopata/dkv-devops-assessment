# Improved Pipeline

```yaml
trigger:
  - main

variables:
  NODE_VERSION: "20"

pool:
  vmImage: ubuntu-22.04

stages:

  - stage: Validate
    jobs:

      - job: Lint
        steps:
          - checkout: self

          - task: Cache@2
            inputs:
              key: 'npm | "$(Agent.OS)" | package-lock.json'
              path: $(Pipeline.Workspace)/.npm

          - script: npm ci
            displayName: Install dependencies

          - script: npm run lint
            displayName: Run ESLint

      - job: UnitTests
        steps:
          - checkout: self

          - task: Cache@2
            inputs:
              key: 'npm | "$(Agent.OS)" | package-lock.json'
              path: $(Pipeline.Workspace)/.npm

          - script: npm ci
            displayName: Install dependencies

          - script: npm test
            displayName: Run unit tests

  - stage: IntegrationTest
    dependsOn: Validate
    jobs:
      - job: IntegrationTests
        steps:
          - checkout: self

          - script: npm ci
            displayName: Install dependencies

          - script: npm run test:integration
            displayName: Run integration tests
            env:
              DB_PASSWORD: $(DB_PASSWORD)

  - stage: Build
    dependsOn: IntegrationTest
    jobs:
      - job: BuildApp
        steps:
          - checkout: self

          - script: npm ci
            displayName: Install dependencies

          - script: npm run build
            displayName: Build application

          - task: PublishBuildArtifacts@1
            inputs:
              pathToPublish: dist
              artifactName: app

  - stage: Deploy
    dependsOn: Build
    jobs:
      - job: DeployProd
        steps:
          - download: current
            artifact: app

          - script: |
              curl -X POST https://deploy.example.com/release \
                -H "Authorization: Bearer $(API_KEY)" \
                -d @dist/manifest.json
            displayName: Deploy to production
```

---

# Improvements Made

- Moved hardcoded secrets to secure pipeline variables
- Replaced `npm install` with `npm ci`
- Added npm dependency cache
- Removed `continueOnError: true`
- Changed `ubuntu-latest` to `ubuntu-22.04`
- Ran lint and unit tests in parallel
- Kept integration tests before build and deploy
- Reused build artifact during deployment

---

# Additional Improvements

If needed, the pipeline could also be improved with:
- staging environment
- smoke tests
- approval before production deployment
- security scanning
- rollback strategy

