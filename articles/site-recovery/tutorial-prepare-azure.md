---
title: Criar recursos para usar com o Azure Site Recovery | Microsoft Docs
description: "Saiba como preparar o Azure para a replicação de máquinas locais usando o serviço Azure Site Recovery."
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: tutorial
ms.date: 01/16/2018
ms.author: raynew
ms.custom: MVC
ms.openlocfilehash: 63290127b298efced14ad34e9223840f3229f046
ms.sourcegitcommit: 7edfa9fbed0f9e274209cec6456bf4a689a4c1a6
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/17/2018
---
# <a name="prepare-azure-resources-for-replication-of-on-premises-machines"></a>Preparar recursos do Azure para replicação de máquinas locais

O serviço [Azure Site Recovery](site-recovery-overview.md) contribui para sua estratégia de BCDR (continuidade de negócios e recuperação de desastre) mantendo seus aplicativos de negócios em execução durante interrupções planejadas e não planejadas. O Site Recovery gerencia e orquestra a recuperação de desastre de máquinas locais e de VMs (máquinas virtuais) do Azure, incluindo replicação, failover e recuperação.

Este tutorial mostra como preparar componentes do Azure quando você deseja replicar VMs locais (Hyper-V ou VMware) ou servidores físicos Windows/Linux para o Azure. Neste tutorial, você aprenderá como:

> [!div class="checklist"]
> * Verifique se sua conta tem permissões de replicação
> * Crie uma conta de armazenamento do Azure.
> * Configure uma rede do Azure. Quando as VMs do Azure são criadas após o failover, elas são ingressadas nessa rede do Azure.

Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de começar.

## <a name="log-in-to-azure"></a>Fazer logon no Azure

Faça logon no Portal do Azure em http://portal.azure.com.

## <a name="verify-account-permissions"></a>Verificar permissões de conta

Se acabou de criar sua conta gratuita do Azure, você é o administrador da assinatura. Se você não for o administrador da assinatura, peça para o administrador atribuir as permissões necessárias. Para habilitar a replicação para uma nova máquina virtual, você deve ter:

- Permissão para criar uma VM no grupo de recursos selecionado
- Permissão para criar uma VM na rede virtual selecionada
- Permissão para gravar na conta de armazenamento selecionada

A função interna 'Colaborador da Máquina Virtual' tem essas permissões. Você também precisa de permissão para gerenciar operações de recuperação do Azure Site Recovery. A função 'Colaborador do Site Recovery' tem todas as permissões necessárias para gerenciar operações do Site Recovery em um cofre de Serviços de Recuperação.

## <a name="create-a-storage-account"></a>Criar uma conta de armazenamento

As imagens de máquinas replicadas são mantidas no armazenamento do Azure. As VMs do Azure são criadas do armazenamento quando há failover do local para o Azure.

1. No menu [Portal do Azure](https://portal.azure.com), clique em **Novo** -> **Armazenamento** -> **Conta de armazenamento**.
2. Em **Criar conta de armazenamento**, insira um nome para a conta. Para esses tutoriais, usaremos o nome **contosovmsacct1910171607**. O nome deve ser exclusivo dentro do Azure e ter entre 3 e 24 caracteres, contendo apenas números e letras minúsculas.
3. Use o modelo de implantação do **Resource Manager**.
4. Selecione **Propósito geral** > **Standard**. Não selecione armazenamento de blobs.
5. Selecione o padrão **RA-GRS** para redundância de armazenamento.
6. Selecione a assinatura na qual você deseja criar a nova conta de armazenamento.
7. Especifique um novo grupo de recursos. Um grupo de recursos do Azure é um contêiner lógico no qual os recursos do Azure são implantados e gerenciados. Para esses tutoriais usamos o nome **ContosoRG**.
8. Selecione a região geográfica para sua conta de armazenamento. A conta de armazenamento deve estar na mesma região do cofre de Serviços de Recuperação. Para estes tutoriais usamos a região **Europa Ocidental**.

   ![create-storageacct](media/tutorial-prepare-azure/create-storageacct.png)

9. Clique em **Criar** para criar a conta de armazenamento.

## <a name="create-a-vault"></a>Criar um cofre

1. No menu do portal do Azure, clique em **Novo** > **Monitoramento e Gerenciamento** >
   **Backup e Site Recovery**.
2. Em **Nome**, especifique um nome amigável para identificar o cofre. Para este tutorial, usamos **ContosoVMVault**.
3. Selecione o grupo de recursos existente denominado **contosoRG**.
4. Especifique a região do Azure **Europa Ocidental**, que estamos usando neste conjunto de tutoriais.
5. Para acessar rapidamente o cofre do painel, clique em **Fixar no painel** > **Criar**.

   ![Novo cofre](./media/tutorial-prepare-azure/new-vault-settings.png)

   O novo cofre será exibido no **Painel** > **Todos os recursos** e na página **Cofres de Serviços de Recuperação** principal.

## <a name="set-up-an-azure-network"></a>Configure uma rede do Azure

Quando as VMs do Azure são criadas do armazenamento após o failover, elas são associadas a esta rede.

1. No menu [Portal do Azure](https://portal.azure.com), clique em **Novo** > **Rede** >
   **Rede Virtual**
2. Deixe **Resource Manager** selecionado como o modelo de implantação. Resource Manager é o modelo de implantação preferencial.
   - Especifique um nome de rede. O nome deve ser exclusivo dentro do grupo de recursos do Azure. Usaremos o nome **ContosoASRnet**
   - Use o grupo de recursos existente **contosoRG**.
   - Especifique o intervalo de endereços de rede 10.0.0.0/24.
   - Para este tutorial, não é necessária uma sub-rede.
   - Selecione a assinatura na qual deseja criar a rede.
   - Selecione o local **Europa Ocidental**. A rede deve estar na mesma região do que o cofre de Recuperação de Site.
3. Clique em **Criar**.

   ![create-network](media/tutorial-prepare-azure/create-network.png)

   A rede virtual leva alguns segundos para ser criada. Depois de ser criada, ela será exibida no painel do Portal do Azure.

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Preparar a infraestrutura do VMware local para recuperação de desastre para o Azure](tutorial-prepare-on-premises-vmware.md)
