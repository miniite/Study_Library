---
type: "practice"
difficulty: "Intermediate"
---

# What is a YAML pipeline file and what are its main sections?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pipeline as Code](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pipeline_as_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A YAML pipeline file, often used in CI/CD systems like Jenkins or GitLab CI, defines the automated build and deployment processes for your application. The key sections include stages, jobs, triggers, and artifacts. This structure ensures that your code changes are systematically tested, built, and deployed.

The question is probing your understanding of how CI/CD pipelines function and what elements they contain to manage the automation process effectively. I'll assume a standard CI/CD pipeline setup where we need to cover all these sections from first principles.

**Code:**

```yaml
# Example GitLab CI/CD YAML file

stages:
  - build
  - test
  - deploy

variables:
  # Define project-wide variables here if needed
  PROJECT_NAME: "my-app"

jobs:
  build:
    stage: build
    script:
      - echo Building $PROJECT_NAME...
      - npm install
      - npm run build

  test:
    stage: test
    dependencies:
      - build
    script:
      - echo Testing $PROJECT_NAME...
      - npm run test

  deploy:
    stage: deploy
    needs: [test]
    environment:
      name: production
      url: "https://$DOMAIN/$PROJECT_NAME"
    when: manual
    script:
      - echo Deploying to production...

triggers:
  - schedule:
      cron: '0 12 * * *'
      branch: master

artifacts:
  paths:
    - build/
  expire_in: 1 week
```

**Flow:**

1. Stages
2. Build
3. Test
4. Deploy
5. Triggers
6. Artifacts

