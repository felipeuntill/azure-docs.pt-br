---
title: "Início rápido do PowerShell no Azure Cloud Shell (versão prévia) | Microsoft Docs"
description: "Início rápido do PowerShell no Cloud Shell"
services: Azure
documentationcenter: 
author: maertendmsft
manager: timlt
tags: azure-resource-manager
ms.assetid: 
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 01/19/2018
ms.author: damaerte
ms.openlocfilehash: 71ae70c13b4de87593345fd957a773741294b49c
ms.sourcegitcommit: 059dae3d8a0e716adc95ad2296843a45745a415d
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/09/2018
---
# <a name="quickstart-for-powershell-in-azure-cloud-shell-preview"></a>Início rápido do PowerShell no Azure Cloud Shell (Versão prévia)

Este documento fornece detalhes sobre como usar o PowerShell no Cloud Shell no [Portal do Azure](https://aka.ms/PSCloudPreview).

> [!NOTE]
> Um início rápido do [Bash no Azure Cloud Shell](quickstart.md) também está disponível.

## <a name="start-cloud-shell"></a>Iniciar o Cloud Shell

1. Clique no botão **Cloud Shell** no painel de navegação superior do portal do Azure

  ![](media/quickstart-powershell/shell-icon.png)

2. Selecione o ambiente do PowerShell na lista suspensa e você irá para unidade do Azure`(Azure:)`

  ![](media/quickstart-powershell/environment-ps.png)

## <a name="run-powershell-commands"></a>Executar comandos do PowerShell

Execute comandos regulares do PowerShell no Cloud Shell, tais como:

```PowerShell
PS Azure:\> Get-Date
Monday, September 25, 2017 08:55:09 AM

PS Azure:\> Get-AzureRmVM -Status

ResourceGroupName       Name       Location                VmSize   OsType     ProvisioningState  PowerState
-----------------       ----       --------                ------   ------     -----------------  ----------
MyResourceGroup2        Demo        westus         Standard_DS1_v2  Windows    Succeeded           running
MyResourceGroup         MyVM1       eastus            Standard_DS1  Windows    Succeeded           running
MyResourceGroup         MyVM2       eastus   Standard_DS2_v2_Promo  Windows    Succeeded           deallocated
```

## <a name="navigate-azure-resources"></a>Navegar por recursos do Azure

 1. Listar suas assinaturas

    ``` PowerShell
    PS Azure:\> dir
    ```

 2. `cd` para sua assinatura preferencial

    ``` PowerShell
    PS Azure:\> cd MySubscriptionName
    PS Azure:\MySubscriptionName>
    ```

 3. Exibir todos os seus recursos do Azure na assinatura atual
 
    Digite `dir` para listar vários modos de exibição de recursos do Azure.
 
    ``` PowerShell
    PS Azure:\MySubscriptionName> dir

        Directory: azure:\MySubscriptionName

    Mode Name
    ---- ----
    +    AllResources
    +    ResourceGroups
    +    StorageAccounts
    +    VirtualMachines
    +    WebApps
     ```

### <a name="allresources-view"></a>Exibição de AllResources 
Digite `dir` no diretório `AllResources` para exibir os recursos do Azure.
    
    PS Azure:\MySubscriptionName> dir AllResources

### <a name="explore-resource-groups"></a>Explorar grupos de recursos

 Você pode ir para o diretório `ResourceGroups` e, dentro de um grupo de recursos específico, é possível encontrar máquinas virtuais.

``` PowerShell
PS Azure:\MySubscriptionName> cd ResourceGroups\MyResourceGroup1\Microsoft.Compute\virtualMachines

PS Azure:\MySubscriptionName\ResourceGroups\MyResourceGroup1\Microsoft.Compute\virtualMachines> dir


    Directory: Azure:\MySubscriptionName\ResourceGroups\MyResourceGroup1\Microsoft.Compute\virtualMachines


VMName    Location   ProvisioningState VMSize          OS            SKU             OSVersion AdminUserName  NetworkInterfaceName
------    --------   ----------------- ------          --            ---             --------- -------------  --------------------
TestVm1   westus     Succeeded         Standard_DS2_v2 WindowsServer 2016-Datacenter Latest    AdminUser      demo371
TestVm2   westus     Succeeded         Standard_DS1_v2 WindowsServer 2016-Datacenter Latest    AdminUser      demo271

```
> [!NOTE]
> Pode-se perceber que ao digitar `dir` pela segunda vez, o Cloud Shell consegue exibir os itens muito mais rapidamente.
> Isso ocorre porque os itens filhos são armazenados em cache na memória para oferecer uma melhor experiência de usuário.
No entanto, você sempre pode usar `dir -Force` para obter dados atualizados.

### <a name="navigate-storage-resources"></a>Navegar por recursos de armazenamento
    
Ao entrar na pasta `StorageAccounts`, você pode navegar facilmente por seus recursos de armazenamento
    
``` PowerShell 
PS Azure:\MySubscriptionName\StorageAccounts\MyStorageAccountName\Files> dir

    Directory: Azure:\MySubscriptionNameStorageAccounts\MyStorageAccountName\Files


Name          ConnectionString
----          ----------------
MyFileShare1  \\MyStorageAccountName.file.core.windows.net\MyFileShare1;AccountName=MyStorageAccountName AccountKey=<key>
MyFileShare2  \\MyStorageAccountName.file.core.windows.net\MyFileShare2;AccountName=MyStorageAccountName AccountKey=<key>
MyFileShare3  \\MyStorageAccountName.file.core.windows.net\MyFileShare3;AccountName=MyStorageAccountName AccountKey=<key>


```

Com a cadeia de conexão, você pode usar o comando a seguir para montar o compartilhamento de arquivos do Azure.
        
``` PowerShell
net use <DesiredDriveLetter>: \\<MyStorageAccountName>.file.core.windows.net\<MyFileShareName> <AccountKey> /user:Azure\<MyStorageAccountName>


```

Para obter detalhes, consulte [Como montar um compartilhamento de arquivos do Azure e acessá-lo no Windows][azmount].

Você também pode navegar pelos diretórios no compartilhamento de arquivos do Azure da seguinte maneira:

            
``` PowerShell
PS Azure:\MySubscriptionName\StorageAccounts\MyStorageAccountName\Files> cd .\MyFileShare1\
PS Azure:\MySubscriptionName\StorageAccounts\MyStorageAccountName\Files\MyFileShare1> dir

Mode  Name
----  ----
+     TestFolder
.     hello.ps1

    
```

### <a name="interact-with-virtual-machines"></a>Interagir com máquinas virtuais

Você pode encontrar todas as suas máquinas virtuais da assinatura atual por meio do diretório `VirtualMachines`.
    
``` PowerShell
PS Azure:\MySubscriptionName\VirtualMachines> dir

    Directory: Azure:\MySubscriptionName\VirtualMachines


Name       ResourceGroupName  Location  VmSize          OsType              NIC ProvisioningState  PowerState
----       -----------------  --------  ------          ------              --- -----------------  ----------
TestVm1    MyResourceGroup1   westus    Standard_DS2_v2 Windows       my2008r213         Succeeded     stopped
TestVm2    MyResourceGroup1   westus    Standard_DS1_v2 Windows          jpstest         Succeeded deallocated
TestVm10   MyResourceGroup2   eastus    Standard_DS1_v2 Windows           mytest         Succeeded     running


```

#### <a name="invoke-powershell-script-across-remote-vms"></a>Invocar o script do PowerShell entre VMs remotas

 > [!WARNING]
 > Consulte [Solucionando problemas de gerenciamento remoto de máquinas virtuais do Azure](troubleshooting.md#powershell-resolutions).

  Supondo que você tenha uma VM, MyVM1, vamos usar `Invoke-AzureRmVMCommand` para invocar um bloco de script do PowerShell no computador remoto.

  ``` Powershell
  Invoke-AzureRmVMCommand -Name MyVM1 -ResourceGroupName MyResourceGroup -Scriptblock {Get-ComputerInfo} -EnableRemoting
  ```
  Você também pode primeiro navegar até o diretório VirtualMachines e executar `Invoke-AzureRmVMCommand` como mostrado a seguir.

  ``` PowerShell
  PS Azure:\> cd MySubscriptionName\MyResourceGroup\Microsoft.Compute\virtualMachines
  PS Azure:\MySubscriptionName\MyResourceGroup\Microsoft.Compute\virtualMachines> Get-Item MyVM1 | Invoke-AzureRmVMCommand -Scriptblock{Get-ComputerInfo}
  ```
  Você verá uma saída semelhante ao seguinte:

  ``` PowerShell
  PSComputerName                                          : 65.52.28.207
  RunspaceId                                              : 2c2b60da-f9b9-4f42-a282-93316cb06fe1
  WindowsBuildLabEx                                       : 14393.1066.amd64fre.rs1_release_sec.170327-1835
  WindowsCurrentVersion                                   : 6.3
  WindowsEditionId                                        : ServerDatacenter
  WindowsInstallationType                                 : Server
  WindowsInstallDateFromRegistry                          : 5/18/2017 11:26:08 PM
  WindowsProductId                                        : 00376-40000-00000-AA947
  WindowsProductName                                      : Windows Server 2016 Datacenter
  WindowsRegisteredOrganization                           :
   ...
  ```

#### <a name="interactively-log-on-to-a-remote-vm"></a>Fazer logon interativamente em uma VM remota

Você pode usar `Enter-AzureRmVM` para fazer logon interativamente em uma VM em execução no Azure.

  ``` PowerShell
  Enter-AzureRmVM -Name MyVM1 -ResourceGroupName MyResourceGroup -EnableRemoting
  ```

Você também pode primeiro navegar até o diretório `VirtualMachines` e executar `Enter-AzureRmVM` como mostrado a seguir

  ``` PowerShell
 PS Azure:\MySubscriptionName\ResourceGroups\MyResourceGroup\Microsoft.Compute\virtualMachines> Get-Item MyVM1 | Enter-AzureRmVM
 ```

### <a name="discover-webapps"></a>Descobrir WebApps

Ao entrar na pasta `WebApps`, você pode navegar facilmente por seus recursos de aplicativo Web

``` PowerShell
PS Azure:\MySubscriptionName> dir .\WebApps\

    Directory: Azure:\MySubscriptionName\WebApps


Name            State    ResourceGroup      EnabledHostNames                  Location
----            -----    -------------      ----------------                  --------
mywebapp1       Stopped  MyResourceGroup1   {mywebapp1.azurewebsites.net...   West US
mywebapp2       Running  MyResourceGroup2   {mywebapp2.azurewebsites.net...   West Europe
mywebapp3       Running  MyResourceGroup3   {mywebapp3.azurewebsites.net...   South Central US



# You can use Azure cmdlets to Start/Stop your web apps
PS Azure:\MySubscriptionName\WebApps> Start-AzureRmWebApp -Name mywebapp1 -ResourceGroupName MyResourceGroup1

Name           State    ResourceGroup        EnabledHostNames                   Location
----           -----    -------------        ----------------                   --------
mywebapp1      Running  MyResourceGroup1     {mywebapp1.azurewebsites.net ...   West US

# Refresh the current state with -Force
PS Azure:\MySubscriptionName\WebApps> dir -Force

    Directory: Azure:\MySubscriptionName\WebApps


Name            State    ResourceGroup      EnabledHostNames                  Location
----            -----    -------------      ----------------                  --------
mywebapp1       Running  MyResourceGroup1   {mywebapp1.azurewebsites.net...   West US
mywebapp2       Running  MyResourceGroup2   {mywebapp2.azurewebsites.net...   West Europe
mywebapp3       Running  MyResourceGroup3   {mywebapp3.azurewebsites.net...   South Central US

```

## <a name="ssh"></a>SSH

[Win32-OpenSSH](https://github.com/PowerShell/Win32-OpenSSH) está disponível no PowerShell Cloud Shell.
Para autenticar servidores ou VMs usando o SSH, gere o par de chaves privadas/públicas no Cloud Shell e publique a chave pública para `authorized_keys` no computador remoto, como `/home/user/.ssh/authorized_keys`.

> [!NOTE]
> É possível criar chaves privadas/públicas SSH usando `ssh-keygen` e publicá-las para `$env:USERPROFILE\.ssh` no Cloud Shell.

### <a name="using-a-custom-profile-to-persist-git-and-ssh-settings"></a>Usando um perfil personalizado para persistir as configurações GIT e SSH

Quando as sessões não persistem após a assinatura, salve sua pasta `$env:USERPROFILE\.ssh` para `CloudDrive` ou crie um quando o Cloud Shell for inicializado.
Adicione o seguinte trecho de código no seu perfil.ps1 para criar um symlink no CloudDrive.

``` PowerShell
# Check if the .ssh folder exists
if( -not (Test-Path $home\CloudDrive\.ssh)){
    mkdir $home\CloudDrive\.ssh
}

# .ssh path relative to this script
$script:sshFolderPath = Join-Path $PSScriptRoot .ssh

# Create a symlink to .ssh in user's $home
if(Test-Path $script:sshFolderPath){
   if(-not (Test-Path (Join-Path $HOME .ssh ))){
        New-Item -ItemType SymbolicLink -Path $HOME -Name .ssh -Value $script:sshFolderPath
   }
}

```

### <a name="using-ssh"></a>Usando o SSH

Siga as instruções [aqui](https://docs.microsoft.com/azure/virtual-machines/linux/quick-create-powershell) para criar uma nova configuração de VM usando cmdlets AzureRM.
Antes de chamar em `New-AzureRmVM` para iniciar a implantação, adicione chave pública SSH à configuração da VM.
A VM recém-criada conterá a chave pública no local `~\.ssh\authorized_keys`, permitindo assim a sessão SSH sem credenciais para a VM.

``` PowerShell

# Create VM config object - $vmConfig using instructions on linked page above

# Generate SSH keys in Cloud Shell
ssh-keygen -t rsa -b 2048 -f $HOME\.ssh\id_rsa 

# Ensure VM config is updated with SSH keys
$sshPublicKey = Get-Content "$env:USERPROFILE\.ssh\id_rsa.pub"
Add-AzureRmVMSshPublicKey -VM $vmConfig -KeyData $sshPublicKey -Path "/home/azureuser/.ssh/authorized_keys"

# Create a virtual machine
New-AzureRmVM -ResourceGroupName <yourResourceGroup> -Location <vmLocation> -VM $vmConfig

# SSH to the VM
ssh azureuser@MyVM.Domain.Com

```


## <a name="list-available-commands"></a>Listar comandos disponíveis

Na unidade `Azure`, digite `Get-AzureRmCommand` para obter comandos do Azure específicos ao contexto.

Como alternativa, você pode sempre usar `Get-Command *azurerm* -Module AzureRM.*` para descobrir os comandos disponíveis do Azure.

## <a name="install-custom-modules"></a>Instalar módulos personalizados

Você pode executar `Install-Module` para instalar módulos da [Galeria do PowerShell][gallery].

## <a name="get-help"></a>Get-Help

Digite `Get-Help` para obter informações sobre o PowerShell no Azure Cloud Shell.

``` PowerShell
PS Azure:\> Get-Help
```

Para um comando específico, você também pode executar `Get-Help` seguido de um cmdlet, por exemplo.

``` PowerShell
PS Azure:\> Get-Help Get-AzureRmVM
```

## <a name="use-azure-files-to-store-your-data"></a>Usar os Arquivos do Azure para armazenar seus dados

Você pode criar um script, digamos `helloworld.ps1`, e salvá-lo em seu `CloudDrive` para usá-lo em sessões de shell.

``` PowerShell
cd C:\users\ContainerAdministrator\CloudDrive
PS C:\users\ContainerAdministrator\CloudDrive> vim .\helloworld.ps1
# Add the content, such as 'Hello World!'
PS C:\users\ContainerAdministrator\CloudDrive> .\helloworld.ps1
Hello World!
```

Na próxima vez que utilizar o PowerShell no Cloud Shell, o arquivo `helloworld.ps1` estará na pasta `CloudDrive` que monta o compartilhamento de arquivos do Azure.

## <a name="use-custom-profile"></a>Usar perfil personalizado

Você pode personalizar o ambiente do PowerShell criando perfis do PowerShell - `profile.ps1` ou `Microsoft.PowerShell_profile.ps1`. Salve-o em `CloudDrive` para que possa ser carregado em toda sessão do PowerShell quando você iniciar o Cloud Shell.

Para saber como criar um perfil, consulte [Sobre perfis][profile].

## <a name="use-git"></a>Usar o Git

Para clonar um repositório Git no Cloud Shell, você precisa criar um [token de acesso pessoal][githubtoken] e usá-lo como nome de usuário. Quando tiver seu token, faça um clone do repositório da seguinte maneira:

 ``` PowerShell
  git clone https://<your-access-token>@github.com/username/repo.git

```
Como as sessões no Cloud Shell não persistem quando você sai ou quando a sessão atinge o tempo limite, o arquivo de configuração do Git não existirá no próximo logon. Para que sua configuração do Git persista, você deve salvar seu .gitconfig em `CloudDrive` e copiá-lo ou criar um symlink quando o Cloud Shell iniciar. Use o seguinte trecho de código em seu profile.ps1 para criar um symlink para `CloudDrive`.

 ``` PowerShell
 
# .gitconfig path relative to this script
$script:gitconfigPath = Join-Path $PSScriptRoot .gitconfig

# Create a symlink to .gitconfig in user's $home
if(Test-Path $script:gitconfigPath){

    if(-not (Test-Path (Join-Path $home .gitconfig ))){
         New-Item -ItemType SymbolicLink -Path $home -Name .gitconfig -Value $script:gitconfigPath
    }
}

```
## <a name="exit-the-shell"></a>Sair do shell

Digite `exit` para encerrar a sessão.

[bashqs]:quickstart.md
[gallery]:https://www.powershellgallery.com/
[customex]:https://docs.microsoft.com/azure/virtual-machines/windows/extensions-customscript
[profile]: https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.core/about/about_profiles
[azmount]: https://docs.microsoft.com/azure/storage/files/storage-how-to-use-files-windows
[githubtoken]: https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/
