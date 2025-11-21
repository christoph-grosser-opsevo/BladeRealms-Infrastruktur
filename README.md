# BladeRealms - Infrastructure Repository

> Azure-Infrastruktur für BladeRealms mit Bicep und PowerShell

[![Azure](https://img.shields.io/badge/Azure-Infrastructure-0078D4.svg)](https://azure.microsoft.com/)
[![Bicep](https://img.shields.io/badge/IaC-Bicep-00A4EF.svg)](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)
[![PowerShell](https://img.shields.io/badge/PowerShell-7+-5391FE.svg)](https://docs.microsoft.com/powershell/)

## 📋 Inhaltsverzeichnis

- [Übersicht](#übersicht)
- [Architektur](#architektur)
- [Prerequisites](#prerequisites)
- [Resource Groups](#resource-groups)
- [Bicep Module](#bicep-module)
- [Deployment](#deployment)
- [Monitoring](#monitoring)
- [Kosten](#kosten)

## 🎯 Übersicht

Dieses Repository enthält die **komplette Azure-Infrastruktur** für BladeRealms als Infrastructure as Code mit Bicep.

**Philosophie:**
- **Dev = Prod**: Identische Infrastruktur-Struktur für alle Umgebungen
- **Bicep-Only**: Keine ARM-Templates, kein Terraform
- **PowerShell-First**: Alle Deployment-Scripts in PowerShell 7+
- **Production-Ready**: Von Tag 1 produktionsreif

## 🏗️ Architektur

```
Azure Subscription: BladeRealms
│
├── Resource Group: rg-bladerealms-dev
│   ├── Azure Container Registry (bladerealmsacrdev)
│   ├── Azure Container Instances (bladerealms-server-dev)
│   ├── Application Insights (ai-bladerealms-dev)
│   └── Log Analytics Workspace (law-bladerealms-dev)
│
├── Resource Group: rg-bladerealms-prod
│   ├── Azure Container Registry (bladerealmsacr)
│   ├── Azure Container Apps (bladerealms-server-prod)
│   ├── Azure Application Gateway (agw-bladerealms)
│   ├── Application Insights (ai-bladerealms-prod)
│   └── Log Analytics Workspace (law-bladerealms-prod)
│
└── Resource Group: rg-bladerealms-shared
    ├── Azure Key Vault (kv-bladerealms)
    └── Storage Account (stbladerealms)
```

## 🔧 Prerequisites

### Tools
```powershell
# PowerShell 7+
winget install Microsoft.PowerShell

# Azure PowerShell Module
Install-Module -Name Az -Repository PSGallery -Force

# Bicep PowerShell Module (optional, in Az.Resources enthalten)
# Bicep wird automatisch mit Az.Resources installiert
```

### Azure Login
```powershell
# Login to Azure
Connect-AzAccount

# Set Subscription
Set-AzContext -SubscriptionName "BladeRealms-Subscription"
```

## 📦 Resource Groups

### Struktur

| Resource Group | Zweck | Umgebung |
|----------------|-------|----------|
| `rg-bladerealms-dev` | Development-Umgebung | Dev |
| `rg-bladerealms-prod` | Production-Umgebung | Prod |
| `rg-bladerealms-shared` | Shared Resources (Key Vault, Storage) | Shared |

### Erstellen

```powershell
# Development
New-AzResourceGroup -Name "rg-bladerealms-dev" -Location "westeurope" -Tag @{Environment="Development"; Project="BladeRealms"}

# Production
New-AzResourceGroup -Name "rg-bladerealms-prod" -Location "westeurope" -Tag @{Environment="Production"; Project="BladeRealms"}

# Shared
New-AzResourceGroup -Name "rg-bladerealms-shared" -Location "westeurope" -Tag @{Environment="Shared"; Project="BladeRealms"}
```

## 🧱 Bicep Module

### Verzeichnisstruktur

```
infrastructure/
├── main.bicep                      # Haupt-Template
├── parameters/
│   ├── dev.parameters.json         # Dev-Parameter
│   └── prod.parameters.json        # Prod-Parameter
├── modules/
│   ├── containerRegistry.bicep     # Azure Container Registry
│   ├── containerInstance.bicep     # Container Instances (Dev)
│   ├── containerApp.bicep          # Container Apps (Prod)
│   ├── applicationGateway.bicep    # Application Gateway
│   ├── keyVault.bicep              # Key Vault
│   └── monitoring.bicep            # App Insights + Log Analytics
└── scripts/
    ├── Deploy-Infrastructure.ps1   # Deployment-Script
    ├── Test-Infrastructure.ps1     # Test-Script
    └── Remove-Infrastructure.ps1   # Cleanup-Script
```

### Bicep Best Practices

```bicep
// Naming Convention
param environment string = 'dev'
param location string = resourceGroup().location
param projectName string = 'bladerealms'

// Tagging
var commonTags = {
  Environment: environment
  Project: projectName
  ManagedBy: 'Bicep'
}

// Outputs
output containerRegistryLoginServer string = containerRegistry.properties.loginServer
output containerInstanceFqdn string = containerInstance.properties.ipAddress.fqdn
```

## 🚀 Deployment

### Development Environment

```powershell
# Navigiere zum Infrastructure-Repository
Set-Location infrastructure

# Validate Bicep Template
Test-AzResourceGroupDeployment `
  -ResourceGroupName "rg-bladerealms-dev" `
  -TemplateFile "main.bicep" `
  -TemplateParameterFile "parameters/dev.parameters.json"

# What-If Analysis
Get-AzResourceGroupDeploymentWhatIfResult `
  -ResourceGroupName "rg-bladerealms-dev" `
  -TemplateFile "main.bicep" `
  -TemplateParameterFile "parameters/dev.parameters.json"

# Deploy
$deploymentName = "deployment-$(Get-Date -Format 'yyyyMMdd-HHmmss')"
New-AzResourceGroupDeployment `
  -ResourceGroupName "rg-bladerealms-dev" `
  -TemplateFile "main.bicep" `
  -TemplateParameterFile "parameters/dev.parameters.json" `
  -Name $deploymentName `
  -Verbose
```

### Production Environment

```powershell
# Validate
Test-AzResourceGroupDeployment `
  -ResourceGroupName "rg-bladerealms-prod" `
  -TemplateFile "main.bicep" `
  -TemplateParameterFile "parameters/prod.parameters.json"

# What-If (WICHTIG vor Prod-Deploy!)
Get-AzResourceGroupDeploymentWhatIfResult `
  -ResourceGroupName "rg-bladerealms-prod" `
  -TemplateFile "main.bicep" `
  -TemplateParameterFile "parameters/prod.parameters.json" `
  | Format-List

# Deploy
$deploymentName = "deployment-$(Get-Date -Format 'yyyyMMdd-HHmmss')"
New-AzResourceGroupDeployment `
  -ResourceGroupName "rg-bladerealms-prod" `
  -TemplateFile "main.bicep" `
  -TemplateParameterFile "parameters/prod.parameters.json" `
  -Name $deploymentName `
  -Verbose
```

### PowerShell Deployment Script

```powershell
# scripts/Deploy-Infrastructure.ps1
[CmdletBinding()]
param(
    [Parameter(Mandatory=$true)]
    [ValidateSet('dev','prod')]
    [string]$Environment
)

$ErrorActionPreference = "Stop"

Write-Host "🚀 Deploying BladeRealms Infrastructure to $Environment..." -ForegroundColor Cyan

$resourceGroupName = "rg-bladerealms-$Environment"
$templateFile = "main.bicep"
$parametersFile = "parameters/$Environment.parameters.json"
$deploymentName = "deployment-$(Get-Date -Format 'yyyyMMdd-HHmmss')"

try {
    # Validate
    Write-Host "✅ Validating Bicep Template..." -ForegroundColor Yellow
    $validation = Test-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile $templateFile `
        -TemplateParameterFile $parametersFile
    
    if ($validation) {
        Write-Error "❌ Validation failed: $($validation.Message)"
        exit 1
    }
    
    Write-Host "✅ Validation successful!" -ForegroundColor Green
    
    # What-If
    Write-Host "🔍 Running What-If Analysis..." -ForegroundColor Yellow
    $whatIf = Get-AzResourceGroupDeploymentWhatIfResult `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile $templateFile `
        -TemplateParameterFile $parametersFile
    
    $whatIf | Format-List
    
    # Confirm Deployment
    $confirm = Read-Host "Continue with deployment? (y/n)"
    if ($confirm -ne 'y') {
        Write-Host "❌ Deployment cancelled" -ForegroundColor Red
        exit 0
    }
    
    # Deploy
    Write-Host "🚀 Deploying..." -ForegroundColor Green
    $deployment = New-AzResourceGroupDeployment `
        -ResourceGroupName $resourceGroupName `
        -TemplateFile $templateFile `
        -TemplateParameterFile $parametersFile `
        -Name $deploymentName `
        -Verbose
    
    Write-Host "✅ Deployment successful!" -ForegroundColor Green
    Write-Host "Deployment Name: $deploymentName" -ForegroundColor Cyan
    
    # Output Results
    $deployment.Outputs.Keys | ForEach-Object {
        Write-Host "$_: $($deployment.Outputs[$_].Value)" -ForegroundColor Yellow
    }
    
} catch {
    Write-Error "❌ Deployment failed: $($_.Exception.Message)"
    exit 1
}
```

## 📊 Monitoring

### Application Insights

```powershell
# Get Application Insights Key
$appInsightsKey = (Get-AzApplicationInsights `
    -ResourceGroupName "rg-bladerealms-prod" `
    -Name "ai-bladerealms-prod").InstrumentationKey

Write-Host "Application Insights Key: $appInsightsKey"
```

### Log Analytics

```powershell
# Query Logs
$workspaceId = (Get-AzOperationalInsightsWorkspace `
    -ResourceGroupName "rg-bladerealms-prod" `
    -Name "law-bladerealms-prod").CustomerId

# Query Container Logs
$query = @"
ContainerInstanceLog_CL
| where TimeGenerated > ago(1h)
| where ContainerGroup_s == "bladerealms-server-prod"
| project TimeGenerated, Message
| order by TimeGenerated desc
"@

Invoke-AzOperationalInsightsQuery -WorkspaceId $workspaceId -Query $query
```

## 💰 Kosten

### Geschätzte monatliche Kosten

| Resource | Dev | Prod (50 CCU) | Prod (100+ CCU) |
|----------|-----|---------------|-----------------|
| Container Registry | Kostenlos (Basic) | €5 (Standard) | €5 |
| Container Instances | €5-10 (Low-Spec) | - | - |
| Container Apps | - | €15-30 | €50-100 |
| Application Gateway | - | €40-60 | €60-100 |
| App Insights | Kostenlos (5GB) | €5-10 | €10-20 |
| Key Vault | €1 | €1 | €1 |
| **Gesamt** | **€6-11/Monat** | **€66-106/Monat** | **€126-226/Monat** |

### Kostenoptimierung

```powershell
# Stop Dev-Environment außerhalb der Arbeitszeiten
# Automation Account mit Runbook

# Get all Container Instances in Dev
$containers = Get-AzContainerGroup -ResourceGroupName "rg-bladerealms-dev"

# Stop Containers
foreach ($container in $containers) {
    Stop-AzContainerGroup -ResourceGroupName $container.ResourceGroupName -Name $container.Name
}
```

## 🔐 Security

### Key Vault Secrets

```powershell
# Set Secrets
Set-AzKeyVaultSecret -VaultName "kv-bladerealms" `
    -Name "PlayFabSecretKey" `
    -SecretValue (ConvertTo-SecureString "YOUR_SECRET" -AsPlainText -Force)

# Get Secrets
$secret = Get-AzKeyVaultSecret -VaultName "kv-bladerealms" -Name "PlayFabSecretKey"
$secretValue = $secret.SecretValue | ConvertFrom-SecureString -AsPlainText
```

## 🧹 Cleanup

```powershell
# Remove Development Environment
Remove-AzResourceGroup -Name "rg-bladerealms-dev" -Force

# Remove Production Environment (VORSICHT!)
Remove-AzResourceGroup -Name "rg-bladerealms-prod" -Force

# Remove Shared Resources
Remove-AzResourceGroup -Name "rg-bladerealms-shared" -Force
```

## 📚 Weitere Ressourcen

- [Azure Container Instances Dokumentation](https://learn.microsoft.com/azure/container-instances/)
- [Azure Container Apps Dokumentation](https://learn.microsoft.com/azure/container-apps/)
- [Bicep Dokumentation](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)
- [PowerShell Azure Module](https://learn.microsoft.com/powershell/azure/)

---

**Repository**: [BladeRealms-Infrastructure](https://github.com/yourusername/bladerealms-infrastructure)
