# 🚀 Azure Pipelines with Azure Resources – YAML Syntax Guide

---

# Azure-Service-Practice
```
az group create --name MyResourceGroup --location eastus

az vm create --resource-group myResourceGroup --name myVM --image Ubuntu2404 --size Standard_B1s --admin-username azureuser --generate-ssh-keys --no-wait
```


## 1️⃣ Azure Service Connection (ARM)

Used to authenticate Azure resources.

```yaml
variables:
  azureSubscription: 'Azure-Service-Connection'
```

Used in tasks:

```yaml
azureSubscription: $(azureSubscription)
```

---

## 2️⃣ Azure CLI Task (Most Common)

### ▶ Run Azure CLI Commands

```yaml
- task: AzureCLI@2
  displayName: Azure CLI Login & Commands
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      az account show
      az group list
```

---

## 3️⃣ Create Resource Group

```yaml
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      az group create \
        --name my-rg \
        --location eastus
```

---

## 4️⃣ Azure Resource Manager (ARM Template)

```yaml
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentScope: Resource Group
    azureResourceManagerConnection: 'Azure-Service-Connection'
    resourceGroupName: my-rg
    location: eastus
    templateLocation: Linked artifact
    csmFile: templates/azuredeploy.json
    csmParametersFile: templates/azuredeploy.parameters.json
    deploymentMode: Incremental
```

---

## 5️⃣ Deploy Azure Web App

```yaml
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    appName: mywebapp
    package: '$(Pipeline.Workspace)/drop/app.zip'
```

---

## 6️⃣ Web App Deployment Slot

```yaml
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    appName: mywebapp
    deployToSlotOrASE: true
    resourceGroupName: myRG
    slotName: staging
    package: '$(Pipeline.Workspace)/drop/app.zip'
```

---

## 7️⃣ Slot Swap (Blue-Green)

```yaml
- task: AzureAppServiceManage@0
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    WebAppName: mywebapp
    ResourceGroupName: myRG
    SourceSlot: staging
    SwapWithProduction: true
```

---

## 8️⃣ Deploy to AKS (kubectl)

```yaml
- task: Kubernetes@1
  inputs:
    connectionType: Azure Resource Manager
    azureSubscriptionEndpoint: 'Azure-Service-Connection'
    azureResourceGroup: aks-rg
    kubernetesCluster: aks-demo
    command: apply
    useConfigurationFile: true
    configuration: manifests/deployment.yaml
```

---

## 9️⃣ Helm Deploy to AKS

```yaml
- task: HelmDeploy@0
  inputs:
    connectionType: Azure Resource Manager
    azureSubscription: 'Azure-Service-Connection'
    azureResourceGroup: aks-rg
    kubernetesCluster: aks-demo
    command: upgrade
    chartType: FilePath
    chartPath: charts/myapp
    releaseName: myapp
```

---

## 🔟 Azure Storage Upload (Blob)

```yaml
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      az storage blob upload \
        --account-name mystorageacct \
        --container-name backups \
        --file app.zip \
        --name app.zip
```

---

## 1️⃣1️⃣ Azure Key Vault Secrets

```yaml
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    KeyVaultName: myKeyVault
    SecretsFilter: '*'
    RunAsPreJob: true
```

Use in pipeline:

```yaml
$(DB_PASSWORD)
```

---

## 1️⃣2️⃣ Terraform with Azure

```yaml
- task: TerraformCLI@0
  inputs:
    command: apply
    environmentServiceName: 'Azure-Service-Connection'
```

---

## 1️⃣3️⃣ Managed Identity Login (No Secret)

```yaml
- task: AzureCLI@2
  inputs:
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      az login --identity
      az group list
```

---

## 1️⃣4️⃣ Conditional Azure Stage

```yaml
- stage: Deploy_Prod
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
```

---

## 1️⃣5️⃣ Full Azure Pipeline Skeleton

```yaml
trigger:
- main

stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - script: echo "Build completed"

- stage: Deploy
  dependsOn: Build
  jobs:
  - job: DeployJob
    steps:
    - task: AzureWebApp@1
      inputs:
        azureSubscription: 'Azure-Service-Connection'
        appName: mywebapp
        package: '$(Pipeline.Workspace)/drop/app.zip'
```

Below is an **expanded & advanced update** of your guide with **more YAML syntaxes, tasks, patterns, and real-world DevOps usage**.
You can directly **append this to the same document**.

---

# 🚀 Azure Pipelines with Azure Resources – **Extended YAML Syntax Guide**

---

## 1️⃣6️⃣ Pipeline Triggers (Advanced)

### ▶ Branch + Path Filters

```yaml
trigger:
  branches:
    include:
      - main
      - develop
  paths:
    include:
      - src/*
    exclude:
      - docs/*
```

### ▶ Disable CI Trigger (Manual / Release Pipelines)

```yaml
trigger: none
```

---

## 1️⃣7️⃣ Variables (Inline, Group, Runtime)

### ▶ Inline Variables

```yaml
variables:
  environment: dev
  location: eastus
```

### ▶ Variable Group (Library → Variable Groups)

```yaml
variables:
- group: Azure-Secrets-Group
```

### ▶ Runtime Variable (Set in Script)

```yaml
- bash: |
    echo "##vso[task.setvariable variable=IMAGE_TAG]$(Build.BuildId)"
```

Usage:

```yaml
$(IMAGE_TAG)
```

---

## 1️⃣8️⃣ Secure Files (Certificates, kubeconfig, .pfx)

```yaml
- task: DownloadSecureFile@1
  name: cert
  inputs:
    secureFile: mycert.pfx
```

Usage:

```yaml
$(cert.secureFilePath)
```

---

## 1️⃣9️⃣ Azure PowerShell Task

```yaml
- task: AzurePowerShell@5
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    ScriptType: InlineScript
    Inline: |
      Get-AzResourceGroup
    azurePowerShellVersion: LatestVersion
```

---

## 2️⃣0️⃣ Deploy Azure VM (CLI)

```yaml
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    scriptType: bash
    scriptLocation: inlineScript
    inlineScript: |
      az vm create \
        --resource-group myRG \
        --name myVM \
        --image Ubuntu2204 \
        --admin-username azureuser \
        --generate-ssh-keys
```

---

## 2️⃣1️⃣ Azure Container Registry (ACR)

### ▶ Login to ACR

```yaml
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    scriptType: bash
    inlineScript: |
      az acr login --name myacr
```

### ▶ Build & Push Image

```yaml
- task: Docker@2
  inputs:
    command: buildAndPush
    repository: myapp
    dockerfile: Dockerfile
    containerRegistry: ACR-Service-Connection
    tags: |
      $(Build.BuildId)
```

---

## 2️⃣2️⃣ AKS – Get Credentials (Explicit)

```yaml
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    scriptType: bash
    inlineScript: |
      az aks get-credentials \
        --resource-group aks-rg \
        --name aks-demo \
        --overwrite-existing
```

---

## 2️⃣3️⃣ AKS – kubectl Inline Commands

```yaml
- task: Kubernetes@1
  inputs:
    command: get
    arguments: pods
```

```yaml
- task: Kubernetes@1
  inputs:
    command: delete
    arguments: pod mypod
```

---

## 2️⃣4️⃣ Helm with Values Override

```yaml
- task: HelmDeploy@0
  inputs:
    command: upgrade
    chartType: FilePath
    chartPath: charts/app
    releaseName: app
    overrideValues: |
      image.tag=$(Build.BuildId)
```

---

## 2️⃣5️⃣ Azure SQL – Firewall Rule

```yaml
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    scriptType: bash
    inlineScript: |
      az sql server firewall-rule create \
        --resource-group myRG \
        --server mysqlserver \
        --name AllowAzure \
        --start-ip-address 0.0.0.0 \
        --end-ip-address 0.0.0.0
```

---

## 2️⃣6️⃣ Azure Function App Deploy

```yaml
- task: AzureFunctionApp@1
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    appType: functionAppLinux
    appName: myfuncapp
    package: '$(Pipeline.Workspace)/drop/function.zip'
```

---

## 2️⃣7️⃣ Bicep Deployment

```yaml
- task: AzureCLI@2
  inputs:
    azureSubscription: 'Azure-Service-Connection'
    scriptType: bash
    inlineScript: |
      az deployment group create \
        --resource-group myRG \
        --template-file main.bicep \
        --parameters env=dev
```

---

## 2️⃣8️⃣ Terraform (Full Flow)

```yaml
- task: TerraformInstaller@1
  inputs:
    terraformVersion: latest
```

```yaml
- task: TerraformCLI@0
  inputs:
    command: init
    environmentServiceName: 'Azure-Service-Connection'
```

```yaml
- task: TerraformCLI@0
  inputs:
    command: plan
```

```yaml
- task: TerraformCLI@0
  inputs:
    command: apply
    args: '-auto-approve'
```

---

## 2️⃣9️⃣ Approvals & Environments

```yaml
environment: production
```

➡ Configure **Manual Approval** in Azure DevOps → Environments.

---

## 3️⃣0️⃣ Multi-Environment Strategy (DEV → QA → PROD)

```yaml
stages:
- stage: Dev
- stage: QA
  dependsOn: Dev
- stage: Prod
  dependsOn: QA
  condition: succeeded()
```

---

## 3️⃣1️⃣ Job Matrix (Multi-OS / Versions)

```yaml
strategy:
  matrix:
    linux:
      imageName: ubuntu-latest
    windows:
      imageName: windows-latest
```

---

## 3️⃣2️⃣ Retry, Timeout & Continue on Error

```yaml
- task: AzureCLI@2
  retryCountOnTaskFailure: 3
  timeoutInMinutes: 10
  continueOnError: true
```

---

## 3️⃣3️⃣ Publish & Download Artifacts

```yaml
- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: drop
    artifactName: drop
```

```yaml
- task: DownloadPipelineArtifact@2
  inputs:
    artifact: drop
```

---

## 🧠 Real-World Usage Mapping

| Scenario       | YAML Feature              |
| -------------- | ------------------------- |
| Blue-Green     | Slots + Swap              |
| Canary         | Traffic Manager + Weights |
| Infra          | Terraform / Bicep         |
| Secure Secrets | Key Vault                 |
| AKS GitOps     | Helm                      |
| Zero Downtime  | Stages + Conditions       |
| Enterprise     | Approvals + Environments  |

---

## 🏆 Enterprise Best Practices (Must-Remember)

✔ Use **Variable Groups + Key Vault**
✔ Prefer **Bicep / Terraform** over portal
✔ Use **Environments with approvals**
✔ Keep **Build & Deploy separated**
✔ Use **conditions instead of multiple pipelines**
✔ Enable **Rollback (slot swap reverse)**

---

## ✅ When to Use What (Quick Guide)

| Resource            | Task                                   |
| ------------------- | -------------------------------------- |
| Resource Group / VM | AzureCLI                               |
| ARM / Bicep         | AzureResourceManagerTemplateDeployment |
| Web App             | AzureWebApp                            |
| Slots               | AzureAppServiceManage                  |
| AKS                 | Kubernetes / HelmDeploy                |
| Secrets             | AzureKeyVault                          |
| Infra as Code       | TerraformCLI                           |

---

## 🔥 Best Practices

✔ Use **Service Connections**
✔ Prefer **Managed Identity**
✔ Separate **Build & Deploy stages**
✔ Use **Key Vault for secrets**
✔ Enable **Branch conditions**
