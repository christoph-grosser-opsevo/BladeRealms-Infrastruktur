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
├── External: GitHub Container Registry (ghcr.io)
│   └── ghcr.io/christoph-grosser-opsevo/bladerealms-server:latest
│
├── Resource Group: rg-bladerealms-dev-weu-001
│   ├── Container Instance: ci-bladerealms-server-dev-weu-001
│   ├── Application Insights: appi-bladerealms-dev-weu
│   └── Log Analytics Workspace: log-bladerealms-dev-weu
│
├── Resource Group: rg-bladerealms-prod-weu-001
│   ├── Container App: ca-bladerealms-server-prod-weu-001
│   ├── Container App Environment: cae-bladerealms-prod-weu
│   ├── Application Gateway: agw-bladerealms-prod-weu
│   ├── Application Insights: appi-bladerealms-prod-weu
│   └── Log Analytics Workspace: log-bladerealms-prod-weu
│
└── Resource Group: rg-bladerealms-shared-weu-001
    ├── Key Vault: kv-bladerealms-shared-weu (inkl. GitHub Registry Credentials)
    ├── Storage Account: stbladereamsshrweu (keine Hyphens erlaubt)
    └── Managed Identity: id-bladerealms-shared-weu

Naming Convention: Microsoft Azure Cloud Adoption Framework (CAF)
Pattern: <resource-type>-<workload>-<environment>-<region>-<instance>
Region-Codes: westeurope=weu, northeurope=neu, eastus2=eus2
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

| Resource Group | Zweck | Umgebung | Naming Pattern |
|----------------|-------|----------|----------------|
| `rg-bladerealms-dev-weu-001` | Development-Umgebung | Dev | rg-<workload>-<env>-<region>-<instance> |
| `rg-bladerealms-prod-weu-001` | Production-Umgebung | Prod | rg-<workload>-<env>-<region>-<instance> |
| `rg-bladerealms-shared-weu-001` | Shared Resources (Key Vault, Storage) | Shared | rg-<workload>-<env>-<region>-<instance> |

### Erstellen

```powershell
# Development
New-AzResourceGroup `
  -Name "rg-bladerealms-dev-weu-001" `
  -Location "westeurope" `
  -Tag @{
    Environment = "Development"
    Project = "BladeRealms"
    ManagedBy = "Bicep"
    Region = "westeurope"
  }

# Production
New-AzResourceGroup `
  -Name "rg-bladerealms-prod-weu-001" `
  -Location "westeurope" `
  -Tag @{
    Environment = "Production"
    Project = "BladeRealms"
    ManagedBy = "Bicep"
    Region = "westeurope"
  }

# Shared
New-AzResourceGroup `
  -Name "rg-bladerealms-shared-weu-001" `
  -Location "westeurope" `
  -Tag @{
    Environment = "Shared"
    Project = "BladeRealms"
    ManagedBy = "Bicep"
    Region = "westeurope"
  }
```

## 🧱 Bicep Module

### Verzeichnisstruktur

```
BladeRealms-Infrastruktur/
├── deployment/
│   ├── Deploy-Infrastructure.ps1   # Haupt-Deployment-Script
│   ├── Deploy-Dev.ps1              # Dev-Deployment
│   ├── Deploy-Prod.ps1             # Prod-Deployment
│   ├── Test-Infrastructure.ps1     # Test-Script
│   └── Remove-Infrastructure.ps1   # Cleanup-Script
│
├── infrastructure/
│   ├── rg-bladerealms-dev-weu-001/
│   │   ├── container-instance.bicepparam
│   │   ├── monitoring.bicepparam
│   │   └── README.md
│   ├── rg-bladerealms-prod-weu-001/
│   │   ├── container-app.bicepparam
│   │   ├── application-gateway.bicepparam
│   │   ├── monitoring.bicepparam
│   │   └── README.md
│   └── rg-bladerealms-shared-weu-001/
│       ├── key-vault.bicepparam
│       ├── storage-account.bicepparam
│       ├── managed-identity.bicepparam
│       └── README.md
│
├── resource/
│   ├── container-instance/
│   │   └── container-instance.bicep
│   ├── container-app/
│   │   ├── container-app.bicep
│   │   └── container-app-environment.bicep
│   ├── application-gateway/
│   │   └── application-gateway.bicep
│   ├── key-vault/
│   │   └── key-vault.bicep
│   ├── storage-account/
│   │   └── storage-account.bicep
│   ├── managed-identity/
│   │   └── managed-identity.bicep
│   └── monitoring/
│       ├── application-insights.bicep
│       └── log-analytics-workspace.bicep
│
└── modules/
    ├── networking/
    │   ├── vnet.bicep
    │   ├── subnet.bicep
    │   └── nsg.bicep
    ├── compute/
    │   └── container-base.bicep
    ├── security/
    │   ├── rbac.bicep
    │   └── managed-identity-base.bicep
    └── monitoring/
        └── diagnostic-settings.bicep

📂 Hierarchie:
deployment/*.ps1
    ↓ (liest Parameter)
infrastructure/rg-*/[resource].bicepparam
    ↓ (using '../../resource/...')
resource/[type]/[resource].bicep
    ↓ (module '../../modules/...')
modules/[type]/[module].bicep
    ↓
Azure Resource Manager
```

### Bicep Best Practices

**Philosophie: Minimalistischer, effizienter Code**

- ✅ **Nur notwendige Ressourcen**: Keine Over-Engineering-Lösungen
- ✅ **DRY-Prinzip**: Module nur bei echter Wiederverwendung
- ✅ **Kurze Beschreibungen**: @description präzise und knapp
- ✅ **Outputs sparsam**: Nur exportieren was benötigt wird

**4-Layer-Architektur:**

1. **deployment/*.ps1** → Orchestriert Deployments (Execution)
2. **infrastructure/rg-*/*.bicepparam** → Parameter + KeyVault-Referenzen (`az.getSecret()`) (Infrastukturbeschreibung in Azure)
3. **resource/[type]/*.bicep** → Ressourcen-Definitionen mit `@secure()` für Secrets (Governance-Schicht)
4. **modules/[type]/*.bicep** → Wiederverwendbare Komponenten (Basis Azure Resource)

**KeyVault-Integration:**

```bicep
// .bicepparam
param registryPassword = az.getSecret('SUB-ID', 'RG', 'KV-NAME', 'SECRET-NAME')

// .bicep
@secure()
param registryPassword string
```

## 🚀 Deployment

### Development Environment

```powershell
# Automatisches Deployment aller Dev-Ressourcen
.\deployment\Deploy-Dev.ps1

# Manuelle Einzelressource (optional)
New-AzResourceGroupDeployment `
  -ResourceGroupName "rg-bladerealms-dev-weu-001" `
  -TemplateParameterFile "infrastructure\rg-bladerealms-dev-weu-001\container-instance.bicepparam" `
  -Verbose
```

### Production Environment

```powershell
# Deployment mit What-If-Prüfung (MANDATORY)
.\deployment\Deploy-Prod.ps1
```

**Wichtig:** What-If-Analyse wird automatisch durchgeführt und erfordert manuelle Bestätigung.

### PowerShell Deployment Script

**Struktur:** Alle `.bicepparam` Files in `infrastructure/rg-*/` werden automatisch durchlaufen.

**Workflow:**
1. Validate → What-If → Confirm → Deploy
2. Iteriert über alle Parameter-Files
3. Outputs werden angezeigt

**Verwendung:**
```powershell
.\deployment\Deploy-Dev.ps1
.\deployment\Deploy-Prod.ps1
```

## 📊 Monitoring

**Application Insights:** Custom Metrics, Player-Count, PvP-Stats, Availability Tests

**Log Analytics:** Container Logs, Kusto Queries, Performance-Dashboards

**Alerts:** Container Restarts, High CPU/Memory, Budget-Überschreitungen

```powershell
# Application Insights Key abrufen
$key = (Get-AzApplicationInsights -ResourceGroupName "rg-name" -Name "appi-name").InstrumentationKey

# Log Analytics Query
Invoke-AzOperationalInsightsQuery -WorkspaceId $workspaceId -Query "ContainerInstanceLog_CL | where TimeGenerated > ago(1h)"
```

## 💰 Kosten

### Geschätzte monatliche Kosten

| Resource | Dev | Prod (50 CCU) | Prod (100+ CCU) |
|----------|-----|---------------|------------------|
| GitHub Container Registry | Kostenlos | Kostenlos | Kostenlos |
| Container Instances | €5-10 (Low-Spec) | - | - |
| Container Apps | - | €15-30 | €50-100 |
| Application Gateway | - | €40-60 | €60-100 |
| App Insights | Kostenlos (5GB) | €5-10 | €10-20 |
| Key Vault | €1 | €1 | €1 |
| **Gesamt** | **€6-11/Monat** | **€61-101/Monat** | **€121-221/Monat** |

**Hinweis**: GitHub Container Registry (ghcr.io) ist für öffentliche Repositories kostenlos und bietet unbegrenzten Speicher.

### Kostenoptimierung

```powershell
# Stop Dev-Environment außerhalb der Arbeitszeiten
# Automation Account mit Runbook

# Get all Container Instances in Dev
$containers = Get-AzContainerGroup -ResourceGroupName "rg-bladerealms-dev-weu-001"

# Stop Containers
foreach ($container in $containers) {
    Write-Host "Stopping Container: $($container.Name)"
    Stop-AzContainerGroup -ResourceGroupName $container.ResourceGroupName -Name $container.Name
}
```

## 🔐 Security

### GitHub Container Registry Setup

**Voraussetzungen:**
1. GitHub Personal Access Token (PAT) mit `read:packages` und `write:packages` Berechtigung
2. Container im GitHub Package Registry: `ghcr.io/christoph-grosser-opsevo/bladerealms-server`

**GitHub Container Registry Credentials in Key Vault speichern:**

```powershell
# GitHub Container Registry Username (GitHub Username)
Set-AzKeyVaultSecret `
    -VaultName "kv-bladerealms-shared-weu" `
    -Name "GithubRegistryUsername" `
    -SecretValue (ConvertTo-SecureString "christoph-grosser-opsevo" -AsPlainText -Force)

# GitHub Container Registry Password (Personal Access Token)
Set-AzKeyVaultSecret `
    -VaultName "kv-bladerealms-shared-weu" `
    -Name "GithubRegistryPassword" `
    -SecretValue (ConvertTo-SecureString "ghp_YourPersonalAccessToken" -AsPlainText -Force)
```

**KeyVault-Referenzen in Bicep:**

**Empfohlen: az.getSecret() in .bicepparam**

```bicep
// .bicepparam
param registryPassword = az.getSecret('SUB-ID', 'RG', 'KV-NAME', 'SECRET-NAME')

// .bicep
@secure()
param registryPassword string
```

**Alternative: PowerShell-Abfrage**

```powershell
$secret = (Get-AzKeyVaultSecret -VaultName "kv-name" -Name "secret").SecretValue
New-AzResourceGroupDeployment ... -registryPassword $secret
```

### Key Vault Secrets Management

**Secret setzen:**
```powershell
Set-AzKeyVaultSecret -VaultName "kv-name" -Name "SecretName" `
  -SecretValue (ConvertTo-SecureString "VALUE" -AsPlainText -Force)
```

**Secret abrufen:**
```powershell
$secret = Get-AzKeyVaultSecret -VaultName "kv-name" -Name "SecretName"
$value = $secret.SecretValue | ConvertFrom-SecureString -AsPlainText
```

**Access Policy:**
```powershell
Set-AzKeyVaultAccessPolicy -VaultName "kv-name" -ObjectId $objectId `
  -PermissionsToSecrets Get,List
```

## 🔑 KeyVault Best Practices

**Secret-Namenskonvention:**
- `GithubRegistryUsername`, `GithubRegistryPassword`
- `PlayFab-Dev-SecretKey`, `PlayFab-Prod-SecretKey`
- Umgebungs-Suffix bei unterschiedlichen Werten

**Deployment-Workflow:**
1. Deploy Shared KeyVault → `Deploy-Shared.ps1`
2. Secrets manuell setzen (einmalig)
3. Deploy Dev/Prod mit `az.getSecret()` Referenzen

**Wichtig:**
- ✅ KeyVault-Referenzen in `.bicepparam`
- ✅ `@secure()` für Secret-Parameter
- ❌ Keine Secrets in Git oder Logs

## 🧹 Cleanup

```powershell
# Remove Development Environment
Remove-AzResourceGroup -Name "rg-bladerealms-dev-weu-001" -Force

# Remove Production Environment (VORSICHT!)
Remove-AzResourceGroup -Name "rg-bladerealms-prod-weu-001" -Force

# Remove Shared Resources
Remove-AzResourceGroup -Name "rg-bladerealms-shared-weu-001" -Force
```

## 📚 Weitere Ressourcen

- [Azure Container Instances Dokumentation](https://learn.microsoft.com/azure/container-instances/)
- [Azure Container Apps Dokumentation](https://learn.microsoft.com/azure/container-apps/)
- [Bicep Dokumentation](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)
- [PowerShell Azure Module](https://learn.microsoft.com/powershell/azure/)

---

**Repository**: [BladeRealms-Infrastructure](https://github.com/yourusername/bladerealms-infrastructure)
