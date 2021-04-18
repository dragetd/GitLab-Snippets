# GitLab-Snippets

Useful snippets for GitLab CI. Not yet sorted in any way.


## SonarQube

Gradle Kotlin syntax without fixed Sonar config in the project, dynamically detecting project name.

```
sonar:
  stage: report
  image:
      name: sonarsource/sonar-scanner-cli:latest
      entrypoint: [ "" ]
  variables:
    SONAR_HOST_URL: "https://sonar.example.com"
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"
    GIT_DEPTH: 0
  script:
    - set -u
    - CI_VERSION=$(sed -n 's/^version *= *"\(.*\)"/\1/p' build.gradle.kts settings.gradle.kts)
    - if [ -z "${SONAR_PROJECT_KEY:-}" ]; then
    -   CI_GROUP=$(sed -n 's/^group *= *"\(.*\)"/\1/p' build.gradle.kts settings.gradle.kts)
    -   CI_NAME=$(sed -n 's/^rootProject.name *= *"\(.*\)"/\1/p' build.gradle.kts settings.gradle.kts)
    -   if [ -z "${CI_GROUP}" ] || [ -z "${CI_NAME}" ]; then
    -     echo "Could not parse project coordinate (use SONAR_PROJECT_KEY to disable autodetection)"; exit 1
    -   fi
    -   CI_PROJECT_KEY=${CI_GROUP}:${CI_NAME}
    -   echo "Detected coordinates ${CI_PROJECT_KEY} (${CI_VERSION})"
    - else
    -   CI_PROJECT_KEY=${SONAR_PROJECT_KEY}
    - fi
    - sonar-scanner -Dsonar.qualitygate.wait=true -Dsonar.login=${SONAR_TOKEN} -Dsonar.projectKey=${CI_PROJECT_KEY} -Dsonar.projectVersion=${CI_VERSION:-"not provided"}
  allow_failure: true
  cache:
    key: sonar
    paths: [ ".sonar/cache" ]
```
