# CI/CD

- [CI/CD](#cicd)
  - [CI](#ci)
  - [CD](#cd)
  - [CI\\CD Tool](#cicd-tool)
    - [CI\\CD Workflow](#cicd-workflow)
    - [CI\\CD Config](#cicd-config)

CI/CD 是一种通过在应用开发阶段引入自动化来频繁向客户交付应用的方法。CI/CD 的核心概念是持续集成、持续交付和持续部署。

![CI/CD的流程](https://www.redhat.com/rhdc/managed-files/ci-cd-flow-desktop.png?cicd=32h281b)

## CI

持续集成（Continuous Integration，CI）是指开发人员在将新提交的代码合并到主分支之前时，需要经过编译和自动化测试流程进行验证。更加准确的来说，CI就是在源代码变更后自动检测、拉取、构建等情况下进行单元测试的过程。

## CD

持续交付（Continuous Delivery）和持续部署（Continuous Deployment）均可以使用CD指代。其中，持续交付是指自动将已验证的代码发布到仓库中的某个特定分支，比如Release。而持续部署则是指自动的将应用发布到生产环境。

## CI\CD Tool

- Jenkins：Jenkins 是一个可扩展的开源 CI/CD 工具，支持广泛的插件生态系统和灵活的配置选项。
- GitLab CI/CD：GitLab 自带了内置的 CI/CD 功能，可以直接在 GitLab 仓库中配置和运行 CI/CD 流程。
- Travis CI：Travis CI 是一个云托管的 CI/CD 平台，广泛用于开源项目。它提供了易于配置和集成的功能。
- GitHub Actions：GitHub Actions 是 GitHub 提供的一种集成 CI/CD 功能的工具。它与 GitHub 仓库紧密集成，可以直接在仓库中编写和运行 CI/CD 流程。

### CI\CD Workflow

### CI\CD Config

```yml
stages:
  - build
  - test

before_script:
  - apt-get update
  - apt-get -y install build-essential cmake

build:
  stage: build
  script:
    - mkdir build
    - cd build
    - cmake ..
    - make

test:
  stage: test
  script:
    - cd build
    - ctest --output-on-failure
```
