---
title: "Introdução ao Apache Kafka - Azure HDInsight | Microsoft Docs"
description: "Saiba como criar um cluster do Apache Kafka no Azure HDInsight. Aprenda a criar tópicos, assinantes e consumidores."
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
ms.assetid: 43585abf-bec1-4322-adde-6db21de98d7f
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: 
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 01/18/2018
ms.author: larryfr
ms.openlocfilehash: 639adb2fdc5a7d76c11397b5027199626a0a4016
ms.sourcegitcommit: b32d6948033e7f85e3362e13347a664c0aaa04c1
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 02/13/2018
---
# <a name="start-with-apache-kafka-on-hdinsight"></a>Começar com o Apache Kafka no HDInsight

Saiba como criar e usar um cluster do [Apache Kafka](https://kafka.apache.org) no Azure HDInsight. O Kafka é uma plataforma de streaming distribuída de software livre que está disponível com o HDInsight. Ela é geralmente usada como um agente de mensagens, pois fornece funcionalidade semelhante a uma fila de mensagens para publicação e assinatura. Kafka geralmente é usado com o Apache Spark e o Apache Storm.

> [!NOTE]
> Atualmente, há duas versões do Kafka disponíveis com o HDInsight: 0.9.0 (HDInsight 3.4) e 0.10.0 (HDInsight 3.5 e 3.6). As etapas neste documento pressupõem que você está usando o Kafka no HDInsight 3.6.

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="create-a-kafka-cluster"></a>Criar um cluster Kafka

Use as seguintes etapas para criar um cluster Kafka no HDInsight:

1. No [Portal do Azure](https://portal.azure.com), selecione **+ Criar um recurso**, **Dados + Análise** e selecione **HDInsight**.
   
    ![Criar um cluster HDInsight](./media/apache-kafka-get-started/create-hdinsight.png)

2. Em **Noções Básicas**, insira as seguintes informações:

    * **Nome do cluster**: o nome do cluster HDInsight.
    * **Assinatura**: selecione a assinatura a ser utilizada.
    * **Nome de usuário de logon do cluster** e **Senha de logon do cluster**: logon ao acessar o cluster por HTTPS. Você pode usar essas credenciais para acessar serviços como a interface do usuário da Web do Ambari ou a API REST.
    * **Nome de usuário do SSH (Secure Shell)**: o logon usado ao acessar o cluster via SSH. Por padrão, a senha é a mesma do logon do cluster.
    * **Grupo de Recursos**: o grupo de recursos para criar o cluster.
    * **Local**: a região do Azure para criar o cluster.

        > [!IMPORTANT]
        > Para alta disponibilidade dos dados, recomendamos selecionar um local (região) que contenha __três domínios de falha__. Para saber mais, veja a seção de [Alta disponibilidade de dados](#data-high-availability).
   
 ![Escolha a assinatura](./media/apache-kafka-get-started/hdinsight-basic-configuration.png)

3. Selecione **Tipo de cluster** e defina os seguintes valores em **Configuração do cluster**:
   
    * **Tipo de Cluster**: Kafka
    * **Versão**: Kafka 0.10.0 (HDI 3.6)

    Por fim, use o botão **Selecionar** para salvar as configurações.
     
 ![Selecione o tipo de cluster](./media/apache-kafka-get-started/set-hdinsight-cluster-type.png)

4. Depois de selecionar o tipo de cluster, use o botão __Selecionar__ para definir o tipo de cluster. Em seguida, use o botão __Avançar__ para concluir a configuração básica.

5. Em **Armazenamento**, selecione ou crie uma Conta de armazenamento. Para as etapas neste documento, deixe os outros campos com os valores padrão. Use o botão __Avançar__ para salvar a configuração de armazenamento.

    ![Definir as configurações de conta de armazenamento do HDInsight](./media/apache-kafka-get-started/set-hdinsight-storage-account.png)

6. Em __Aplicativos (opcionais)__, selecione __Avançar__ para continuar. Nenhum aplicativo é necessário neste exemplo.

7. Em __Tamanho do cluster__, selecione __Avançar__ para continuar.

    > [!WARNING]
    > Para garantir a disponibilidade do Kafka no HDInsight, o cluster deve conter pelo menos três nós de trabalho. Para saber mais, veja a seção de [Alta disponibilidade de dados](#data-high-availability).

    ![Definir o tamanho do cluster Kafka](./media/apache-kafka-get-started/kafka-cluster-size.png)

    > [!IMPORTANT]
    > A entrada dos **discos por nó de trabalho** controla a escalabilidade do Kafka no HDInsight. O Kafka no HDInsight usa o disco local das máquinas virtuais no cluster. Como o Kafka tem E/S bastante pesadas, os [Discos Gerenciados do Azure](../../virtual-machines/windows/managed-disks-overview.md) são usados para fornecer a alta taxa de transferência e fornecer mais armazenamento por nó. O tipo de disco gerenciado pode ser __Standard__ (HDD) ou __Premium__ (SSD). Os discos Premium são usados com as VMs das séries DS e GS. Todos os outros tipos VM usam o padrão.

8. Em __Configurações avançadas__, selecione __Avançar__ para continuar.

9. Em **Resumo**, examine a configuração do cluster. Use os links __Editar__ para alterar as configurações que estão incorretas. Por fim, use o botão __Criar__ para criar o cluster.
   
    ![Resumo da configuração do cluster](./media/apache-kafka-get-started/hdinsight-configuration-summary.png)
   
    > [!NOTE]
    > Pode levar até 20 minutos para criar o cluster.

## <a name="connect-to-the-cluster"></a>Conectar-se ao cluster

> [!IMPORTANT]
> Ao executar as etapas a seguir, você deve usar um cliente SSH. Para saber mais, consulte o documento [Usar SSH com HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

No cliente, use o SSH para se conectar ao cluster:

```ssh SSHUSER@CLUSTERNAME-ssh.azurehdinsight.net```

Substitua **SSHUSER** pelo nome de usuário SSH fornecido durante a criação do cluster. Substitua **CLUSTERNAME** pelo nome do cluster.

Quando solicitado, digite a senha usada para a conta SSH.

Para obter informações, consulte [Usar SSH com HDInsight](../hdinsight-hadoop-linux-use-ssh-unix.md).

## <a id="getkafkainfo"></a>Obter as informações de host do Zookeeper e Broker

Ao trabalhar com Kafka, você deve saber os dois valores de host; os hosts *Zookeeper* e os hosts *Broker*. Esses hosts são usados com a API Kafka e muitos dos utilitários fornecidos com o Kafka.

Use as etapas a seguir para criar variáveis de ambiente que contêm as informações de host. As variáveis de ambiente são usadas nas etapas deste documento.

1. Em uma conexão SSH ao cluster, use o comando a seguir para instalar o utilitário `jq`. Esse utilitário é usado para analisar documentos JSON e é útil para recuperar as informações do host de agente:
   
    ```bash
    sudo apt -y install jq
    ```

2. Para definir as variáveis de ambiente com as informações recuperadas do Ambari, use os seguintes comandos:

    ```bash
    CLUSTERNAME='your cluster name'
    export KAFKAZKHOSTS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/ZOOKEEPER/components/ZOOKEEPER_SERVER | jq -r '["\(.host_components[].HostRoles.host_name):2181"] | join(",")' | cut -d',' -f1,2`

    export KAFKABROKERS=`curl -sS -u admin -G https://$CLUSTERNAME.azurehdinsight.net/api/v1/clusters/$CLUSTERNAME/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2`

    echo '$KAFKAZKHOSTS='$KAFKAZKHOSTS
    echo '$KAFKABROKERS='$KAFKABROKERS
    ```

    > [!IMPORTANT]
    > Defina `CLUSTERNAME=` para o nome do cluster do Kafka. Quando solicitado, insira a senha para a conta de logon do cluster (admin).

    O seguinte texto é um exemplo do conteúdo de `$KAFKAZKHOSTS`:
   
    `zk0-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181,zk2-kafka.eahjefxxp1netdbyklgqj5y1ud.ex.internal.cloudapp.net:2181`
   
    O seguinte texto é um exemplo do conteúdo de `$KAFKABROKERS`:
   
    `wn1-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092,wn0-kafka.eahjefxxp1netdbyklgqj5y1ud.cx.internal.cloudapp.net:9092`

    > [!NOTE]
    > O comando `cut` é usado para cortar a lista de hosts em duas entradas de host. Você não precisa fornecer a lista completa de hosts ao criar um consumidor ou produtor do Kafka.
   
    > [!WARNING]
    > Não tome como certo que as informações retornadas nessa sessão sempre são precisas. Se você dimensionar o cluster, novos agentes serão adicionados ou removidos. Se ocorrer uma falha e um nó for substituído, o nome do host para o nó poderá ser alterado.
    >
    > Você deve recuperar as informações de hosts Zookeeper e de agente logo antes de usá-los para garantir que tenha informações válidas.

## <a name="create-a-topic"></a>Criar um tópico

O Kafka armazena fluxos de dados em categorias chamadas *tópicos*. Em uma conexão SSH a um nó principal de cluster, use um script fornecido com o Kafka para criar um tópico:

```bash
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic test --zookeeper $KAFKAZKHOSTS
```

Esse comando conecta ao Zookeeper usando as informações de host armazenadas em `$KAFKAZKHOSTS` e cria um tópico do Kafka chamado **teste**. Você pode verificar se o tópico foi criado usando o seguinte script para listar tópicos:

```bash
/usr/hdp/current/kafka-broker/bin/kafka-topics.sh --list --zookeeper $KAFKAZKHOSTS
```

A saída desse comando lista os tópicos do Kafka, que contém o tópico **teste**.

## <a name="produce-and-consume-records"></a>Produzir e consumir registros

O Kafka armazena *registros* nos tópicos. Os registros são produzidos por *produtores* e consumidos por *consumidores*. Os produtores produzem registros para os *agentes* kafka. Cada nó de trabalho no cluster HDInsight é um agente do Kafka.

Use as seguintes etapas para armazenar registros no tópico teste criado anteriormente e lê-los usando um consumidor:

1. Na sessão SSH, use um script fornecido com o Kafka para gravar registros no tópico:
   
    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list $KAFKABROKERS --topic test
    ```
   
    Você não é levado ao prompt após esse comando. Em vez disso, digite algumas mensagens de texto e use **Ctrl + C** para deixar de enviar ao tópico. Cada linha é enviada como um registro separado.

2. Use um script fornecido com o Kafka para ler registros do tópico:
   
    ```bash
    /usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --bootstrap-server $KAFKABROKERS --topic test --from-beginning
    ```
   
    Esse comando recupera os registros do tópico e os exibe. O uso de `--from-beginning` instrui o consumidor a começar do início do fluxo, para que todos os registros sejam recuperados.

    > [!NOTE]
    > Se você estiver usando uma versão mais antiga do Kafka, talvez seja necessário substituir `--bootstrap-server $KAFKABROKERS` por `--zookeeper $KAFKAZKHOSTS`.

3. Use __Ctrl + C__ para interromper o consumidor.

Você também pode criar programaticamente produtores e consumidores. Para obter um exemplo de como usar essa API, consulte o documento [API do Produtor e Consumidor Kafka com HDInsight](apache-kafka-producer-consumer-api.md).

## <a name="data-high-availability"></a>Alta disponibilidade de dados

Cada região do Azure (local) fornece _domínios de falha_. Um domínio de falha é um agrupamento lógico de hardware subjacente em um data center do Azure. Cada domínio de falha tem um comutador de rede e uma fonte de alimentação em comum. As máquinas virtuais e os discos gerenciados que implementam os nós em um cluster HDInsight são distribuídos entre esses domínios de falha. Essa arquitetura limita o possível impacto de falhas físicas de hardware.

Para obter informações sobre o número de domínios de falha em uma região, consulte o documento [Disponibilidade de máquinas virtuais do Linux](../../virtual-machines/windows/manage-availability.md#use-managed-disks-for-vms-in-an-availability-set).

> [!IMPORTANT]
> É recomendável usar uma região do Azure que contenha três domínios de falha e um fator de replicação de 3.

Se você precisa usar uma região que contém apenas dois domínios de falha, use um fator de replicação de 4 para distribuir as réplicas uniformemente entre os dois domínios de falha.

### <a name="kafka-and-fault-domains"></a>Kafka e domínios de falhas

O Kafka não está ciente dos domínios de falha. Durante a criação de réplicas da partição para tópicos, ele não poderá distribuir réplicas corretamente para alta disponibilidade. Para garantir a alta disponibilidade, use a [ferramenta de rebalanceio de partição Kafka](https://github.com/hdinsight/hdinsight-kafka-tools). Essa ferramenta deve ser executada em uma sessão SSH para o nó principal do cluster Kafka.

Para garantir a mais alta disponibilidade de seus dados do Kafka, você deve rebalancear as réplicas de partição do tópico nas seguintes situações:

* Quando um novo tópico ou uma partição é criado

* Quando você expande um cluster

## <a name="delete-the-cluster"></a>Excluir o cluster

[!INCLUDE [delete-cluster-warning](../../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="troubleshoot"></a>Solucionar problemas

Se você tiver problemas com a criação de clusters HDInsight, confira os [requisitos de controle de acesso](../hdinsight-administer-use-portal-linux.md#create-clusters).

## <a name="next-steps"></a>Próximas etapas

Neste documento, você aprendeu os fundamentos do trabalho com o Apache Kafka no HDInsight. Confira o seguinte para obter mais informações sobre como trabalhar com o Kafka:

* [Analisar logs do Kafka](apache-kafka-log-analytics-operations-management.md)
* [Replicar dados entre clusters de Kafka](apache-kafka-mirroring.md)
* [API do Produtor e Consumidor Kafka com HDInsight](apache-kafka-producer-consumer-api.md)
* [API de Streams do Kafka com HDInsight](apache-kafka-streams-api.md)
* [Use streaming do Apache Spark (DStream) com o Kafka no HDInsight](../hdinsight-apache-spark-with-kafka.md)
* [Use fluxo estruturado do Apache Spark com o Kafka no HDInsight](../hdinsight-apache-kafka-spark-structured-streaming.md)
* [Usar o streaming estruturado do Apache Spark para mover dados do Kafka no HDInsight para o Cosmos DB](../apache-kafka-spark-structured-streaming-cosmosdb.md)
* [Usar Apache Storm com Kafka no HDInsight](../hdinsight-apache-storm-with-kafka.md)
* [Conectar-se ao Kafka por meio de uma Rede Virtual do Azure](apache-kafka-connect-vpn-gateway.md)
* [Usar o Kafka com o Serviço de Contêiner do Azure](apache-kafka-azure-container-services.md)
* [Usar o Kafka com Aplicativos de funções do Azure](apache-kafka-azure-functions.md)
