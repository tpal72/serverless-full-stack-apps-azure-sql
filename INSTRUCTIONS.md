(Invoke-WebRequest -Uri "https://ipinfo.io/ip").Content

# Collect password

$adminSqlLogin = "dbadmin"
$password = Read-Host "Your username is 'cloudadmin'. Please enter a password for your Azure SQL Database server that meets the password requirements"
// dbpassword01!

# Prompt for local ip address

$ipAddress = Read-Host "Disconnect your VPN, open PowerShell on your machine and run '(Invoke-WebRequest -Uri "https://ipinfo.io/ip").Content'. Please enter the value (include periods) next to 'Address': "

Write-Host "Password and IP Address stored"

# Get resource group and location and random string

$resourceGroupName = "tp-svrless-01-rg"
$location = "northeurope"

New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location

$resourceGroup = Get-AzResourceGroup | Where ResourceGroupName -like $resourceGroupName
$uniqueID = Get-Random -Minimum 100000 -Maximum 1000000
// 854676

# The logical server name has to be unique in the system

$serverName = "bus-server$($uniqueID)"

# The sample database name

$databaseName = "bus-db"  
Write-Host "Please note your unique ID for future exercises in this module:"  
Write-Host $uniqueID
Write-Host "Your resource group name is:"
Write-Host $resourceGroupName
Write-Host "Your resources were deployed in the following region:"
Write-Host $location
Write-Host "Your server name is:"
Write-Host $serverName

# storage account
$storageAccountName = $("storageaccount$($uniqueID)")
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroupName -AccountName $storageAccountName -Location $location -SkuName Standard_GRS

$ctx = $storageAccount.Context

$containerName = "bus"
New-AzStorageContainer -Name $containerName -Context $ctx -Permission blob

# Create a new server with a system wide unique server name
$server = New-AzSqlServer -ResourceGroupName $resourceGroupName `
    -ServerName $serverName `
    -Location $location `
    -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminSqlLogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))

# Create a server firewall rule that allows access from the specified IP range and all Azure services
$serverFirewallRule = New-AzSqlServerFirewallRule `
    -ResourceGroupName $resourceGroupName `
    -ServerName $serverName `
    -FirewallRuleName "AllowedIPs" `
    -StartIpAddress $ipAddress -EndIpAddress $ipAddress 

$allowAzureIpsRule = New-AzSqlServerFirewallRule `
    -ResourceGroupName $resourceGroupName `
    -ServerName $serverName `
    -AllowAllAzureIPs

# Create a database
$database = New-AzSqlDatabase  -ResourceGroupName $resourceGroupName `
    -ServerName $serverName `
    -DatabaseName $databaseName `
    -Edition "GeneralPurpose" -Vcore 4 -ComputeGeneration "Gen5" `
    -ComputeModel Serverless -MinimumCapacity 0.5

Write-Host "Database deployed."


# DB CONNECTION STRING
Server=bus-server854676.database.windows.net,1433;Initial Catalog=bus-db;User Id=dbadmin;Password=dbpassword01!;Connection Timeout=30;

# GITHUB TOKEN
vscode://vscode.github-authentication/did-authenticate?windowid=1&code=9bdaa6760fcd9344e24f&state=8ae6e916-c543-40cf-8c3c-62fbbdddcea2

- Copy the token.
- Switch back to VS code.
- Click Signing in to github.com... in the status bar.
- Paste the token and hit enter.

# Azure function name
$azureFunctionName = $("azfunc$($uniqueID)")

$functionApp = New-AzFunctionApp -Name $azureFunctionName `
    -ResourceGroupName $resourceGroupName -StorageAccount $storageAccountName `
    -FunctionsVersion 3 -RuntimeVersion 3 -Runtime dotnet -Location $location
