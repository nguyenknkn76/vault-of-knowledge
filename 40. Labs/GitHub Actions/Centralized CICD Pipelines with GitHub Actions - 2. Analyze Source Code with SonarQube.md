---
tags:
  - devops
  - cicd
  - cicd/git
  - git
  - sonarqube
---
```yml
# path: application-repo/.github/workflows/pr-opened.yml
name: PULL REQUEST OPENED

on:
  pull_request:
    branches: [main, develop]
    types: [opened, synchronize]

jobs:
  code_quality_check:
    uses: DOP2025/centralized-cicd-pipeline/.github/workflows/reusable-pr-opened.yml@main
    secrets:
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_PROJECT_KEY: 'shopsquare-frontend'
```

```yaml
# path: centralized-pipeline-repo/.github/workflows/reusable-pr-opened.yml
name:  Reusable Pull Request Opened

on:
  workflow_call:
    secrets:
      SONAR_HOST_URL:
        required: false
      SONAR_TOKEN:
        required: false
      SONAR_PROJECT_KEY:
        required: false

jobs:
  demo:
    runs-on: dop10_self_hosted_runner_01
    steps:
      - name: Checkout Caller Repository Code
        uses: actions/checkout@v4

      - name: Checkout Centralized Actions
        uses: actions/checkout@v4
        with:
          repository: DOP2025/centralized-cicd-pipeline
          path: centralized-cicd-pipeline

      - name: Detect Programing Language of Project
        id: detect_lang
        uses: ./centralized-cicd-pipeline/.github/actions/detect-language

      - name: Analyze Source Code with SonarQube
        uses: ./centralized-cicd-pipeline/.github/actions/analyze-code
        with:
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_PROJECT_KEY: ${{ secrets.SONAR_PROJECT_KEY }}
```

```yml
# path: centralized-pipeline-repo/.github/actions/analyze-code/action.yml
name: 'SonarQube Code Analysis'
description: 'A composite action to run SonarQube analysis on a project.'

inputs:
  SONAR_HOST_URL:
    description: 'The URL of the SonarQube server.'
    required: true
  SONAR_TOKEN:
    description: 'The authentication token for the SonarQube server.'
    required: true
  SONAR_PROJECT_KEY:
    description: 'The unique key for the project in SonarQube.'
    required: true
  PROJECT_BASE_DIR:
    description: 'The base directory of the project to be analyzed.'
    required: false
    default: '.'

runs:
  using: 'composite'
  steps:
    # The SonarScanner CLI, which this action uses, requires Java to be installed.
    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: SonarQube Scan
      uses: sonarsource/sonarqube-scan-action@master
      env:
        SONAR_TOKEN: ${{ inputs.SONAR_TOKEN }}
        SONAR_HOST_URL: ${{ inputs.SONAR_HOST_URL }}
      with:
        projectBaseDir: ${{ inputs.PROJECT_BASE_DIR }}
        args: >
          -Dsonar.projectKey=${{ inputs.SONAR_PROJECT_KEY }}
```

