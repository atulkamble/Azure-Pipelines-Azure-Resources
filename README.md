# 🚀 Azure Pipelines with Azure Resources – YAML Syntax Guide

---

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
