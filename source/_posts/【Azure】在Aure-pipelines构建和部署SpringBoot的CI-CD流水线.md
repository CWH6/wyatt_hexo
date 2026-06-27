---
title: 【Azure】在Aure pipelines构建和部署SpringBoot的CI/CD流水线
date: 2024-07-31 23:53:23
tags:
  - Azure
  - CICD
  - pipelines
category: 
  - 运维
---

## Azure

Azure 是微软推出的`云计算服务平台`，提供了一系列的`云服务`，包括`计算、分析、存储和网络等`。用户可以利用这些服务`开发、测试、部署和管理应用程序`。Azure 支持多种编程语言、工具和框架，使其适用于各种需求，从简单的云应用到复杂的企业级解决方案。其核心优势在于高可用性、灵活性和安全性，帮助企业降低IT成本，提高运营效率。Azure 还`支持混合云环境，允许企业将本地数据中心与云服务无缝集成`。

## Pipelines

Azure Pipelines 是`Azure DevOps 服务`中的一部分，用于`实现持续集成和持续部署（CI/CD）`。它支持多种编程语言和项目类型，通过定义构建和发布管道，帮助开发团队自动化代码编译、测试和部署流程。Azure Pipelines 可以集成 GitHub、Azure Repos 等源码管理工具，确保代码变更能迅速且可靠地发布到生产环境。其核心功能包括并行构建、云托管的构建代理、丰富的任务库和扩展支持，使其成为 DevOps 实践中不可或缺的工具，有助于提高开发效率和软件质量。

## 场景

当我们springboot项目中，假设有`三个分支（测试，正式，外服）`，`每个分支`的代码 分别部署在`三个服务器`上。

为了实现自动化部署，我们可以利用 azure pipelines 为每个分支设置触发器，并为每个分支配置不同的部署步骤。

> 注意： 每个分支的配置文件内容不应该相同

## 操作

> 注意：下方操作未经过验证，一种 纸上谈兵

### 注册azure账号，创建项目

地址： https://dev.azure.com/ ，构建项目步骤（Repos）略过

### 项目创建分支

点击左边的 `Repos` ，再点击 `Branches` , 再点击 `New branch`

[![image-20240801232006844](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240801232006844.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240801232006844.png)

创建分支取名 `test`, `prod`, `foreign`

### 分支配置文件

对应分支 `test`, `prod`, `foreign` 的 application.yaml 对应的环境调整

### 创建 Azure Pipelines

点击 `Pipelines` 点击 下面 `Pipelines`, 再次点击 `Create Pipelines`

[![image-20240801232524580](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240801232524580.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240801232524580.png)

选择`Azure Repos Git`

[![image-20240801232555407](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240801232555407.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240801232555407.png)

选择下面

[![image-20240801232856032](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240801232856032.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240801232856032.png)

这时候他会帮我们创建一个pipeline yaml 文件

[![image-20240801233014326](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240801233014326.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240801233014326.png)

### 将密码存储到Azure Key Vault

将test服务器密码存到Azure Key Vault，避免在yaml中泄漏

### 将ssh私钥上传到Azure DevOps

在 Azure DevOps 项目库的“Pipelines” -> “Library”中创建一个安全文件，并上传你的 SSH 私钥。如：prod服务器，foreign服务器

### 编写pipeline yaml

假设情况如下：

```
test分支 ->  test服务器 1111.1111.1111.1111  密码：123456    需要将打包的jar 放到 /test/jars

prod分支 ->  prod服务器 1112.1112.1112.1112  ssh私钥文件登陆  需要将打包的jar 放到 /prod/jars

foreign分支 ->  foreign服务器 1113.1113.1113.1113  ssh私钥文件登陆  需要将打包的jar 放到 /foreign/jars
```

yaml 文件如下：

```yaml
trigger:
  branches:
    include:
      - test
      - prod
      - foreign

pool:
  vmImage: 'ubuntu-latest'  # 使用最新的 Linux 版本

variables:
  TEST_SERVER: '1111.1111.1111.1111'
  PROD_SERVER: '1112.1112.1112.1112'
  FOREIGN_SERVER: '1113.1113.1113.1113'
  TEST_DEPLOY_PATH: '/test/jars'
  PROD_DEPLOY_PATH: '/prod/jars'
  FOREIGN_DEPLOY_PATH: '/foreign/jars'
  TEST_PASSWORD: '123456'
  PROD_SSH_KEY: 'prod-server-ssh-key'  # 上传到 Azure DevOps 的 SSH 私钥文件名称
  FOREIGN_SSH_KEY: 'foreign-server-ssh-key'  # 上传到 Azure DevOps 的 SSH 私钥文件名称

steps:
- task: AzureKeyVault@1
  inputs:
    azureSubscription: '<Azure-DevOps-Service-Connection>'
    KeyVaultName: '<Your-KeyVault-Name>'
    SecretsFilter: 'TEST_PASSWORD,PROD_SSH_KEY,FOREIGN_SSH_KEY'
    RunAsPreJob: true

- task: DownloadSecureFile@1
  inputs:
    secureFile: '$(PROD_SSH_KEY)'
    displayName: 'Download SSH Key for Production Server'

- task: DownloadSecureFile@1
  inputs:
    secureFile: '$(FOREIGN_SSH_KEY)'
    displayName: 'Download SSH Key for Foreign Server'

- script: |
    mkdir -p ~/.ssh
    mv $(Agent.TempDirectory)/$(PROD_SSH_KEY) ~/.ssh/id_rsa_prod
    chmod 600 ~/.ssh/id_rsa_prod
    mv $(Agent.TempDirectory)/$(FOREIGN_SSH_KEY) ~/.ssh/id_rsa_foreign
    chmod 600 ~/.ssh/id_rsa_foreign
  displayName: 'Setup SSH Keys for Production and Foreign Servers'

- task: Gradle@2
  inputs:
    workingDirectory: ''
    gradleWrapperFile: 'gradlew'
    gradleOptions: '-Xmx3072m'
    javaHomeOption: 'JDKVersion'
    jdkVersionOption: '1.8'
    jdkArchitectureOption: 'x64'
    publishJUnitResults: true
    testResultsFiles: '**/TEST-*.xml'
    tasks: 'build -x test'

- script: ./gradlew test
  displayName: 'Run tests'
  continueOnError: true

- task: PublishTestResults@2
  inputs:
    testResultsFiles: '**/build/test-results/test/TEST-*.xml'
    searchFolder: '$(System.DefaultWorkingDirectory)'
    testRunTitle: 'Gradle Unit Tests'

- task: CopyFiles@2
  inputs:
    contents: '**/build/libs/*.jar'
    targetFolder: '$(Build.ArtifactStagingDirectory)'

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'

- ${{ if eq(variables['Build.SourceBranchName'], 'test') }}:
  - script: |
      echo "Deploying to Test Server"
      sshpass -p '$(TEST_PASSWORD)' scp $(Build.ArtifactStagingDirectory)/*.jar root@$(TEST_SERVER):$(TEST_DEPLOY_PATH)
      sshpass -p '$(TEST_PASSWORD)' ssh root@$(TEST_SERVER) "cd $(TEST_DEPLOY_PATH) && java -jar *.jar &"
    displayName: 'Deploy to Test Server'

- ${{ if eq(variables['Build.SourceBranchName'], 'foreign') }}:
  - script: |
      echo "Deploying to Foreign Server"
      scp -i ~/.ssh/id_rsa_foreign $(Build.ArtifactStagingDirectory)/*.jar root@$(FOREIGN_SERVER):$(FOREIGN_DEPLOY_PATH)
      ssh -i ~/.ssh/id_rsa_foreign root@$(FOREIGN_SERVER) "cd $(FOREIGN_DEPLOY_PATH) && java -jar *.jar &"
    displayName: 'Deploy to Foreign Server'

- ${{ if eq(variables['Build.SourceBranchName'], 'prod') }}:
  - script: |
      echo "Deploying to Production Server"
      scp -i ~/.ssh/id_rsa_prod $(Build.ArtifactStagingDirectory)/*.jar root@$(PROD_SERVER):$(PROD_DEPLOY_PATH)
      ssh -i ~/.ssh/id_rsa_prod root@$(PROD_SERVER) "cd $(PROD_DEPLOY_PATH) && java -jar *.jar &"
    displayName: 'Deploy to Production Server'
```

##### 测试分支流水线—会话运行jar

上面那个存在问题，每个环境都应该有各种piplines yaml，下面是测试环境的（密码暂时放到yaml上）

部署的流程：

提交代码到test分支 —> 触发test分支流水线 —> 构建gradle依赖 (第一次要很久) —> 构建jar ,推送到服务器指定目录上 —> 创建ts_api会话 —> 在ts_api会话中运行该jar

pipeline yaml

```yaml
trigger:
  branches:
    include:
      - test

pool:
  vmImage: 'ubuntu-latest'

variables:
  TEST_SERVER: '111.111.1111.1111'
  TEST_DEPLOY_PATH: '/root/api/jars'
  TEST_USER: 'root'
  TEST_PASSWORD: '123456'
  JAR_NAME: 'ts_api.jar'
  SESSION_NAME: 'ts_api'

steps:
- task: Gradle@2
  inputs:
    workingDirectory: ''
    gradleWrapperFile: 'gradlew'
    gradleOptions: '-Xmx3072m'
    javaHomeOption: 'JDKVersion'
    jdkVersionOption: '1.8'
    jdkArchitectureOption: 'x64'
    publishJUnitResults: true
    testResultsFiles: '**/TEST-*.xml'
    tasks: 'build -x test'

- script: ./gradlew test
  displayName: 'Run tests'
  continueOnError: true

- task: PublishTestResults@2
  inputs:
    testResultsFiles: '**/build/test-results/test/TEST-*.xml'
    searchFolder: '$(System.DefaultWorkingDirectory)'
    testRunTitle: 'Gradle Unit Tests'

- task: CopyFiles@2
  inputs:
    contents: '**/build/libs/*.jar'
    targetFolder: '$(Build.ArtifactStagingDirectory)'
    flattenFolders: true
    overWrite: true

- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'

- script: |
    echo "Deploying to Test Server"
    sshpass -p '$(TEST_PASSWORD)' scp -o StrictHostKeyChecking=no $(Build.ArtifactStagingDirectory)/*.jar $(TEST_USER)@$(TEST_SERVER):$(TEST_DEPLOY_PATH)/$(JAR_NAME)
    sshpass -p '$(TEST_PASSWORD)' ssh -o StrictHostKeyChecking=no $(TEST_USER)@$(TEST_SERVER) <<EOF
      echo "Stopping existing jar"
      screen -S $(SESSION_NAME) -p 0 -X stuff "^C"
      sleep 5
      echo "Starting new jar"
      screen -dmS $(SESSION_NAME) java -jar $(TEST_DEPLOY_PATH)/$(JAR_NAME)
    EOF
  displayName: 'Deploy and Restart Service on Test Server'
```

流水线构建过程，可以`提交代码触发`或者`手动跑流水线 触发` 更新自动化部署

[![image-20240806223230644](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240806223230644.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240806223230644.png)

**验证**

访问swagger-ui, 或者看服务器上的jar 的更新时间 或者jar的日志 ，或者看会话列表去找ts_api

会话列表，如果没有对应的会话，就说明jar存在问题

```shell
screen -ls
```

进入test_api会话, 是jar的运行控制台

```shell
screen -r 998675.ts_api
```

**注意**

> 存在问题，如果jar有问题，但是流水线会通过，所以需要本地去测试一下jar能不能正常运行

##### **测试分支流水线—dockerfile运行jar**
