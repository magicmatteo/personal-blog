+++ 
draft = false
date = 2025-03-08
title = "Function App Cleanup - Automated"
description = "Delete function apps and all supporting resources via script"
slug = ""
authors = ["Matthew Macdonald"]
tags = ["reference", "azure", "script", "automation", "powershell", "functionApps"]
categories = []
externalLink = ""
series = []
+++

# Overview

I've found myself needing to clean up function apps after migrating them over to new infra from time to time. For 
example, you might be migrating some unmanaged infra into IaC in a new landing zone and need to delete the existing
infra. The annoying thing about this is having to manually track down the storage account that is associated with the 
function app.. A smart person would generally add a tag to the FA that contains the name of the storage account - but
this is seldom the case with legacy, un-IaC'd infra.

So I wrote a script that automates it.. It will grab the name of the storage account from one of the env vars that
contains it. It will then delete the function app, storage account and app service plan (if its empty after FA deletion).

```pwsh
param (
    [string]$rg
)

if (-not $rg) {
    Write-Host "Usage: .\Delete-FunctionApps.ps1 -rg <your-resource-group>" -ForegroundColor Yellow
    exit 1
}

Write-Host "RESOURCE GROUP = $rg"

Write-Host "Fetching function apps...."
$functionApps = az functionapp list --resource-group $rg --query "[].{name:name, plan:appServicePlanId}" --output json | ConvertFrom-Json

if ($functionApps.Count -eq 0) {
    Write-Host "No function apps found in resource group '$rg'." -ForegroundColor Green
    exit 0
}

# track resources to delete
$appsToDelete = @()
$plansToDelete = @{}
$storageAccountsToDelete = @()


foreach ($app in $functionApps) {
    $appName = $app.name
    $planId = $app.plan

    $appsToDelete += $appName

    Write-Host "Fetching storage account name for $appName"
    $storageAccountConnString = az functionapp config appsettings list --name $appName --resource-group $rg --query "[?name=='AzureWebJobsStorage'].value" --output json | ConvertFrom-Json

    if ($storageAccountConnString) {
        if ($storageAccountConnString -match "AccountName=([^;]+)") {
            $accountName = $matches[1]
            $storageAccountsToDelete += $accountName
        }
    }

    if ($planId) {
        $planName = ($planId -split '/')[-1]
        if (-not $plansToDelete.ContainsKey($planName)) {
            $plansToDelete[$planName] = $planId
        }
    }
}

# filter app service plans: only delete if they are empty
$emptyPlansToDelete = @()
foreach ($plan in $plansToDelete.Keys) {
    $planId = $plansToDelete[$plan]

    Write-Host "Fetching app service plan: $plan"
    $azPlan = az appservice plan show --ids $planId --output json | ConvertFrom-Json
        
    if ($azPlan.Properties.numberOfSites -eq 1) {
        # only the one we're deleting
        $emptyPlansToDelete += $plan
    }
}

# print resources to be deleted
Write-Host "`nResources to be deleted:" -ForegroundColor Yellow
Write-Host "---------------------------------" -ForegroundColor Yellow
Write-Host "Function Apps:" -ForegroundColor Cyan
$appsToDelete | ForEach-Object { Write-Host "  - $_" }
Write-Host "Storage Accounts:" -ForegroundColor Cyan
$storageAccountsToDelete | ForEach-Object { Write-Host "  - $_" }
Write-Host "App Service Plans (if empty):" -ForegroundColor Cyan
$emptyPlansToDelete | ForEach-Object { Write-Host "  - $_" }
Write-Host "---------------------------------" -ForegroundColor Yellow

# confirm deletion
$confirmation = Read-Host "Are you sure you want to delete these resources? (yes/no)"
if ($confirmation -ne "yes") {
    Write-Host "Aborting deletion." -ForegroundColor Red
    exit 1
}

# delete functionapps
foreach ($app in $appsToDelete) {
    Write-Host "Deleting function app: $app..." -ForegroundColor Green
    az functionapp delete --name $app --resource-group $rg
}


# delete empty app service plans
foreach ($plan in $emptyPlansToDelete) {
    Write-Host "Deleting empty app service plan: $plan..." -ForegroundColor Green
    az appservice plan delete --name $plan --resource-group $rg --yes
}

# delete storage accounts
$storageAccountsToDelete | ForEach-Object -Parallel {
    Write-Host "Deleting storage account: $PSItem..." -ForegroundColor Green
    az storage account delete --name $PSItem --resource-group $USING:rg --yes
} -ThrottleLimit 5

Write-Host "Cleanup complete!" -ForegroundColor Green
```