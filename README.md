# gradle-demo-with-docker

The following is how to use jenkins CI/CD pipeline to build and release a gradle application via docker.

The repo contains Jenkinsfile, ansible playbook, Java application, gradle configurations, shell scripts and etc.

## Architecture

![Gradle-demo-with-docker architecture](./docs/Gradle-demo-with-docker.png)

## Procedure

![Pipeline steps](./docs/blueocean-1.png)

```txt
Checkout source code  ==> Functional test ==> Build and release image  ==> Initialize test env ==> Deploy test env via ansible ==> Health Check in test env ==> Initialize prod env ==> Deploy prod env via ansible ==> Health Check in prod env
```

Remember to proactive the prerequisite before rollout the pipeline.

## Prerequisite

```txt
Github ===> Git repo

Jenkins ===> CI/CD system

Docker ===> Docker container

Java ===> Java programming language

Gradle ===> Java Build tool

Nexus ===> Binary Package Repo

Python ===> Ansible dependency

Ansible ===> Deployment tool
```

## Runbook

1. Build all prerequisites in an instance or more instances depend on your budget

2. Create an [ansible docker](https://github.com/showerlee/gradle-demo-with-docker/tree/master/ansible) in Jenkins node.

3. Create a jenkins pipeline item

4. Setup jenkins plugin:

    ```txt
    Multibranch Scan Webhook Trigger
    Blue Ocean
    AnsiColor
    ```

5. Provision current repo in pipeline setting with proper credentials

6. Rock and roll

## Test

### Run locally

- Startup the gradle demo in local

  ```bash
  ./auto/start-local-dev
  ```

- Stop the gradle demo in local

  ```bash
  ./auto/stop-local-dev
  ```

### Local/Functional test

Run the following command for local functional test, this would be helpful to test demo before commit a change.

```bash
./auto/run-functional-test
```

This test will be run as a mandatory check in the pipeline before we really build the image.
