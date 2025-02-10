# GitHub Actions Notes

This cheat sheet provides a quick reference for setting up GitHub Actions workflows for building, testing, deploying, and pushing artifacts (e.g., Docker images, build artifacts) to various registries including Artifactory. It also includes detailed notes and best practices.

---

## Overview

GitHub Actions allows you to automate your workflows directly within your GitHub repository. Workflows are defined in YAML files under the `.github/workflows/` directory and can be triggered by events like pushes, pull requests, or on a schedule.

---

## Notes and Best Practices

### 1. Secrets Management
- Never hardcode sensitive information in your workflows. Use GitHub Secrets to store API keys, passwords, and other sensitive data.
- Regularly rotate secrets to maintain security.
- Use environment variables to pass secrets within workflows securely.

### 2. Workflow Triggers
- Customize the `on` section to trigger workflows on events that match your development process (e.g., `push`, `pull_request`, or `schedule`).
- Use `workflow_dispatch` to manually trigger workflows when needed.
- Schedule workflows using cron expressions for automated tasks.

### 3. Versioning Actions
- Always specify versions for actions (e.g., `actions/checkout@v2`) to avoid unexpected changes.
- Check the action’s repository for updates and breaking changes.
- Pin dependencies to specific versions to ensure reproducibility.

### 4. Caching Dependencies
- Utilize caching strategies (such as [`actions/cache`](https://github.com/actions/cache)) to speed up build times by caching dependencies.
- For Docker builds, use build cache layers to optimize image creation.
- Cache Maven or NPM dependencies to improve CI/CD speed.

### 5. Error Handling and Notifications
- Consider adding steps for notifications (e.g., Slack, email) to alert your team if a build or deployment fails.
- Use `continue-on-error: true` for non-blocking failures when necessary.
- Implement retries for transient errors in deployment steps.

### 6. Matrix Builds
- If your project needs to run tests on multiple operating systems or environments, consider using matrix builds to cover all scenarios in a single workflow file.
- Example:
  ```yaml
  strategy:
    matrix:
      os: [ubuntu-latest, windows-latest]
      node: [14, 16]
  ```
- This ensures compatibility across different configurations.

### 7. Documentation and Maintenance
- Keep your workflows documented to ensure easy maintenance.
- Use meaningful workflow names and job descriptions.
- Archive outdated workflows and remove deprecated actions.
- Maintain a version history for critical workflows.

### 8. Security Best Practices
- Restrict workflow execution to specific branches using:
  ```yaml
  on:
    push:
      branches:
        - main
  ```
- Use branch protection rules to prevent unauthorized changes.
- Ensure runners are patched and secure by updating dependencies regularly.

### 9. Efficient Artifact Management
- Use `actions/upload-artifact` and `actions/download-artifact` to manage artifacts between jobs efficiently.
- Store artifacts in Artifactory, AWS S3, or other external storage for persistence.

### 10. Debugging Workflows
- Enable debug mode in GitHub Actions by setting:
  ```yaml
  env:
    ACTIONS_RUNNER_DEBUG: true
    ACTIONS_STEP_DEBUG: true
  ```
- Check workflow logs under GitHub Actions for troubleshooting.
- Use `echo` statements or `set -x` for debugging shell scripts.

### 11. Optimizing Docker Builds
- Use multi-stage builds to reduce the final image size.
- Minimize the number of layers in Dockerfiles.
- Use `.dockerignore` to exclude unnecessary files from the image.
- Utilize `--cache-from` to speed up Docker builds.

### 12. Parallelizing Jobs
- Run independent jobs in parallel using:
  ```yaml
  jobs:
    job1:
      runs-on: ubuntu-latest
    job2:
      runs-on: ubuntu-latest
  ```
- Use job dependencies to sequence execution:
  ```yaml
  needs: job1
  ```

### 13. Ensuring Idempotency
- Ensure deployment scripts can be safely re-run without causing unintended side effects.
- Use rollback mechanisms in case of failures.

### 14. Using Conditionals
- Control job execution based on conditions:
  ```yaml
  if: github.ref == 'refs/heads/main'
  ```
- Use GitHub context variables to dynamically handle logic within workflows.

### 15. Environment-Specific Workflows
- Separate workflows for different environments (e.g., development, staging, production).
- Use environment variables and secrets for environment-specific configurations.
- Example:
  ```yaml
  env:
    APP_ENV: production
  ```


