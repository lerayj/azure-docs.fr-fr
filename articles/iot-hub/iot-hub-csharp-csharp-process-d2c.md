---
title: Traiter les messages des appareils vers le cloud IoT Hub (.Net) | Microsoft Docs
description: Suivez ce didacticiel pour découvrir des modèles utiles pour traiter les messages des appareils vers le cloud IoT Hub.
services: iot-hub
documentationcenter: .net
author: dominicbetts
manager: timlt
editor: ''

ms.service: iot-hub
ms.devlang: csharp
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/05/2016
ms.author: dobett

---
# <a name="tutorial-how-to-process-iot-hub-devicetocloud-messages-using-net"></a>Didacticiel : traiter les messages des appareils vers le cloud IoT Hub à l’aide de .Net
[!INCLUDE [iot-hub-selector-process-d2c](../../includes/iot-hub-selector-process-d2c.md)]

## <a name="introduction"></a>Introduction
Azure IoT Hub est un service entièrement géré qui autorise des communications bidirectionnelles fiables et sécurisées entre des millions d’appareils IoT et un serveur d’applications principal. D’autres didacticiels ([Prise en main d’IoT Hub] et [Envoi de messages cloud-à-appareil avec IoT Hub][lnk-c2d]) expliquent comment utiliser les fonctionnalités de base de messagerie appareil-à-cloud et cloud-à-appareil d’IoT Hub.

Ce didacticiel s’appuie sur le code illustré dans le didacticiel [Prise en main d’IoT Hub] et décrit deux modèles évolutifs que vous pouvez utiliser pour le traitement des messages de l’appareil vers le cloud :

* Le stockage fiable des messages envoyés de l’appareil vers le cloud dans le [stockage d’objets blob Azure]. Un scénario courant est l’analyse de *chemin à froid* , dans laquelle vous stockez les données de télémétrie dans des objets Blob qui sont ensuite utilisés comme entrée dans les processus d’analyse. Ces processus peuvent être pilotés par des outils tels qu’[Azure Data Factory] ou la pile [HDInsight (Hadoop)].
* Le traitement fiable des messages *interactifs* des appareils vers le cloud. Les messages envoyés de l’appareil vers le cloud sont considérés comme interactifs lorsqu’ils agissent comme déclencheurs immédiats d’un ensemble d’actions sur le système principal de l’application. Par exemple, un appareil peut envoyer un message d’alerte qui déclenche l’insertion d’un ticket dans un système CRM. Par opposition, les messages de *point de données* sont simplement chargés dans un moteur d’analyse. Par exemple, les données de télémétrie de température d’un appareil qui doivent être enregistrées pour analyse ultérieure constituent un message de point de données.

Étant donné qu’IoT Hub expose un point de terminaison compatible avec [Event Hubs][lnk-event-hubs] pour recevoir des messages appareil-à-cloud, ce didacticiel utilise une instance [EventProcessorHost]. Cette instance :

* stocke de manière fiable les messages de *point de données* dans le stockage d’objets blob Azure ;
* transfère les messages *interactifs* appareil-à-cloud vers une [file d’attente Azure Service Bus] en vue de leur traitement immédiat.

Service Bus permet d’assurer un traitement fiable des messages interactifs, car il fournit des points de contrôle par message et la déduplication basée sur la fenêtre temporelle.

> [!NOTE]
> Une instance **EventProcessorHost** ne constitue qu’une seule façon de traiter les messages interactifs. Les autres options incluent [Azure Service Fabric][lnk-service-fabric] et [Azure Stream Analytics][lnk-stream-analytics].
> 
> 

À la fin de ce didacticiel, vous exécutez trois applications de console Windows :

* **SimulatedDevice**, une version modifiée de l’application créée dans le didacticiel [Prise en main de IoT Hub] qui envoie des messages de point de données des appareils vers le cloud chaque seconde et des messages interactifs des appareils vers le cloud toutes les 10 secondes. Cette application utilise le protocole AMQPS pour communiquer avec IoT Hub.
* **ProcessDeviceToCloudMessages** utilise la classe [EventProcessorHost] pour récupérer des messages à partir du point de terminaison compatible avec Event Hubs. L’instance stocke ensuite les messages de point de données dans le stockage d’objets Blob Azure de façon fiable, puis transfère les messages interactifs à une file d’attente Service Bus.
* **ProcessD2CInteractiveMessages** extrait les messages interactifs de la file d’attente Service Bus.

> [!NOTE]
> IoT Hub offre la prise en charge de Kit de développement logiciel (SDK) de plusieurs plateformes d’appareils et de plusieurs langages (notamment C, Java et JavaScript). Pour savoir comment remplacer l’appareil simulé de ce didacticiel par un appareil physique, et comment connecter des appareils à IoT Hub, voir le [Centre de développement Azure IoT].
> 
> 

Ce didacticiel s’applique directement à d’autres manières de consommer des messages compatibles avec Event Hubs, par exemple des projets [HDInsight (Hadoop)] . Pour plus d’informations, consultez le [Guide du développeur Azure IoT Hub - Appareil vers cloud].

Pour réaliser ce didacticiel, vous avez besoin des éléments suivants :

* Microsoft Visual Studio 2015.
* Un compte Azure actif. <br/>Si vous n’avez pas d’abonnement Azure, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/free/) en quelques minutes.

Vous devez avoir une connaissance de base de [Stockage Azure] et d’[Azure Service Bus].

## <a name="send-interactive-messages-from-a-simulated-device"></a>Envoi de messages interactifs depuis un appareil simulé
Dans cette section, vous modifiez l’application de l’appareil simulé que vous avez créée dans le didacticiel [Prise en main d’IoT Hub] pour envoyer des messages appareil-à-cloud interactifs au IoT Hub.

1. Dans Visual Studio, dans le projet **SimulatedDevice**, ajoutez la méthode suivante à la classe **Program**.
   
    ```
    private static async void SendDeviceToCloudInteractiveMessagesAsync()
    {
      while (true)
      {
        var interactiveMessageString = "Alert message!";
        var interactiveMessage = new Message(Encoding.ASCII.GetBytes(interactiveMessageString));
        interactiveMessage.Properties["messageType"] = "interactive";
        interactiveMessage.MessageId = Guid.NewGuid().ToString();
   
        await deviceClient.SendEventAsync(interactiveMessage);
        Console.WriteLine("{0} > Sending interactive message: {1}", DateTime.Now, interactiveMessageString);
   
        Task.Delay(10000).Wait();
      }
    }
    ```
   
    Cette méthode est similaire à la méthode **SendDeviceToCloudMessagesAsync** dans le projet **SimulatedDevice**. Les seules différences sont que, désormais, vous définissez la propriété système **MessageId** et une propriété utilisateur nommée **messageType**.
    Le code assigne un identificateur global unique (GUID) à la propriété **MessageId** . Service Bus peut l’utiliser pour dédupliquer les messages qu’il reçoit. L’exemple utilise la propriété **messageType** pour distinguer les messages interactifs et les messages de point de données. L’application transmet ces informations dans les propriétés du message, plutôt que dans le corps du message, afin que le processeur d’événements n’ait pas besoin de désérialiser le message pour le router.
   
   > [!NOTE]
   > Il est important de créer le **MessageId** utilisé pour dédupliquer les messages interactifs dans le code de l’appareil. Les communications réseau intermittentes ou d’autres défaillances pourraient entraîner une retransmission multiple du même message à partir de cet appareil. Vous pouvez également utiliser un ID de message sémantique, comme un hachage des champs de données de message pertinents, à la place d’un GUID.
   > 
   > 
2. Ajoutez la méthode suivante à la méthode **Main** juste avant la ligne `Console.ReadLine()` :
   
    ````
    SendDeviceToCloudInteractiveMessagesAsync();
    ````
   
   > [!NOTE]
   > Par souci de simplicité, ce didacticiel n’implémente aucune stratégie de nouvelle tentative. Dans le code de production, vous devez mettre en œuvre des stratégies de nouvelle tentative (par exemple, une interruption exponentielle), comme indiqué dans l’article MSDN [Transient Fault Handling](Gestion des erreurs temporaires.md).
   > 
   > 

## <a name="process-devicetocloud-messages"></a>Traitement des messages appareil-à-cloud
Dans cette section, vous créez une application de console Windows qui traite les messages appareil-à-cloud à partir d’IoT Hub. IoT Hub expose un point de terminaison compatible avec [Event Hubs]pour permettre à une application de lire les messages appareil-à-cloud. Ce didacticiel utilise la classe [EventProcessorHost] pour traiter ces messages dans une application de console. Pour plus d’informations sur la façon de traiter les messages à partir des concentrateurs d’événements, reportez-vous au didacticiel [Prise en main des concentrateurs d’événements] .

Le défi auquel vous êtes confronté lors de l’implémentation d’un stockage fiable des messages de point de données ou du transfert des messages interactifs, est que le traitement des événements s’appuie sur le consommateur de message pour contrôler la progression. En outre, pour atteindre un débit élevé, lors de la lecture à partir d’Event Hubs, vous devez effectuer les contrôles par grands lots, créant ainsi une possibilité de traitement en double pour un grand nombre de messages en cas d’erreur où vous devriez revenir au point de contrôle précédent. Ce didacticiel explique comment synchroniser des écritures dans le Stockage Azure et des fenêtres de déduplication Service Bus avec les points de contrôle **EventProcessorHost**.

Pour écrire des messages dans le Stockage Azure de manière fiable, l’exemple utilise la fonctionnalité de validation de bloc des [objets blob de blocs][Objets blob de blocs Azure]. Le processeur d’événements accumule les messages en mémoire jusqu'à ce qu’il soit temps de fournir un point de contrôle. Par exemple, une fois que la mémoire tampon de messages accumulés atteint la taille de bloc maximale de 4 Mo, ou après le délai d’expiration de la déduplication Service Bus. Ensuite, avant d’effectuer le contrôle, le code valide un nouveau bloc dans l’objet blob.

Le processeur d’événements utilise les décalages des messages des hubs d’événements comme ID de bloc. Ce mécanisme permet au processeur d’événements d’effectuer une vérification de la déduplication avant de valider le nouveau bloc dans le stockage, en tenant compte d’un possible incident entre un bloc et le point de contrôle.

> [!NOTE]
> Ce didacticiel utilise un même compte de stockage pour enregistrer tous les messages récupérés d’IoT Hub. Pour décider si vous avez besoin d’utiliser plusieurs comptes Azure Storage dans votre solution, consultez [Objectifs de performance et évolutivité d’Azure Storage].
> 
> 

L’application utilise la fonctionnalité de déduplication de Service Bus afin d’éviter les doublons lorsqu’elle traite les messages interactifs. L’appareil simulé affecte à chaque message interactif un **MessageId**unique. Ces ID permettent à Service Bus de garantir que la fenêtre de temps de déduplication spécifiée ne remettra jamais deux messages possédant le même **MessageId** aux récepteurs. Cette déduplication, conjointement avec la sémantique d’achèvement par message fournie par les files d’attente Service Bus, facilite le traitement fiable des messages interactifs.

Pour vous assurer qu’aucun message n’est renvoyé en dehors de la fenêtre de déduplication, le code synchronise le mécanisme de point de contrôle **EventProcessorHost** avec la fenêtre de déduplication de la file d’attente Service Bus. Pour cette synchronisation, vous devez forcer un point de contrôle au moins une fois par écoulement de fenêtre de déduplication (une heure dans ce didacticiel).

> [!NOTE]
> Ce didacticiel utilise une file d’attente Service Bus partitionnée unique pour traiter tous les messages interactifs récupérés d’IoT Hub. Pour plus d’informations sur l’utilisation des files d’attente Service Bus pour répondre aux exigences d’évolutivité de votre solution, consultez la documentation [Azure Service Bus] .
> 
> 

### <a name="provision-an-azure-storage-account-and-a-service-bus-queue"></a>Approvisionner un compte Azure Storage et une file d’attente Service Bus
Pour utiliser la classe [EventProcessorHost] , vous devez disposer d’un compte Azure Storage pour permettre à **l’EventProcessorHost** d’enregistrer les informations de point de contrôle. Vous pouvez utiliser un compte de stockage existant ou suivre les instructions figurant dans [À propos des comptes de stockage Azure] pour en créer un. Notez la chaîne de connexion du compte de stockage.

> [!NOTE]
> Lorsque vous copiez et collez la chaîne de connexion du compte de stockage, assurez-vous que la chaîne de connexion ne contient aucun espace.
> 
> 

Vous avez également besoin d’une file d’attente Service Bus pour permettre un traitement fiable des messages interactifs. Vous pouvez créer une file d’attente par programmation avec une fenêtre de déduplication de 1 heure, comme expliqué dans [Prise en main des files d’attente Service Bus][file d’attente Azure Service Bus]. Vous pouvez également utiliser le [portail Azure Classic][portail classique d’un lien], en procédant comme suit :

1. Cliquez sur **Nouveau** dans l’angle inférieur gauche. Cliquez ensuite sur **App Services** > **Service Bus** > **File d’attente** > **Création personnalisée**. Entrez le nom **d2ctutorial**, sélectionnez une région et utilisez un espace de noms existant ou créez-en un. Dans la page suivante, sélectionnez **Activer la détection des doublons**, puis définissez la **fenêtre d’heures de l’historique des détections dupliquées** sur une heure. Cliquez ensuite sur la coche dans l’angle inférieur droit pour enregistrer la configuration de votre file d’attente.
   
    ![Créer une file d’attente dans le portail Azure][30]
2. Dans la liste des files d’attente Service Bus, cliquez sur **d2ctutorial**, puis sur **Configurer**. Créez deux stratégies d’accès partagé, l’une appelée **send** avec les autorisations **Send**, et l’autre nommée **listen** avec les autorisations **Listen**. Lorsque vous avez terminé, cliquez sur **Enregistrer** en bas.
   
    ![Configurer une file d’attente dans le portail Azure][31]
3. Cliquez sur **Tableau de bord** en haut, puis sur **Informations** de connexion en bas. Notez les deux chaînes de connexion.
   
    ![Tableau de bord de la file d’attente du portail Azure][32]

### <a name="create-the-event-processor"></a>Créer le processeur d’événements
1. Dans la solution Visual Studio actuelle, pour créer un projet Windows Visual C# à l’aide du modèle de projet d’**application console**, cliquez sur **Fichier** > **Ajouter** > **Nouveau projet**. Assurez-vous que la version du .NET Framework est définie sur 4.5.1 ou supérieur. Nommez le projet **ProcessDeviceToCloudMessages**, puis cliquez sur **OK**.
   
    ![Nouveau projet dans Visual Studio][10]
2. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **ProcessDeviceToCloudMessages**, puis cliquez sur **Gérer les packages NuGet**. La boîte de dialogue **Gestionnaire de packages NuGet** apparaît.
3. Recherchez **WindowsAzure.ServiceBus**, cliquez sur **Installer**, puis acceptez les conditions d’utilisation. Cette opération télécharge et installe le [package NuGet Azure Service Bus](https://www.nuget.org/packages/WindowsAzure.ServiceBus), et ajoute une référence à celui-ci, avec toutes ses dépendances.
4. Recherchez **Microsoft.Azure.ServiceBus.EventProcessorHost**, cliquez sur **Installer**, puis acceptez les conditions d’utilisation. Cette opération télécharge et installe le [package NuGet Hub d’événements Azure Service Bus - EventProcessorHost](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost), et ajoute une référence à celui-ci, avec toutes ses dépendances.
5. Cliquez avec le bouton droit sur le projet **ProcessDeviceToCloudMessages**, cliquez sur **Ajouter**, puis sur **Classe**. Nommez la nouvelle classe **StoreEventProcessor**, puis cliquez sur **OK** pour créer la classe.
6. Ajoutez les instructions ci-après au début du fichier StoreEventProcessor.cs :
   
    ```
    using System.IO;
    using System.Diagnostics;
    using System.Security.Cryptography;
    using Microsoft.ServiceBus.Messaging;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    ```
7. Insérez le code suivant dans le corps de la classe :
   
    ```
    class StoreEventProcessor : IEventProcessor
    {
      private const int MAX_BLOCK_SIZE = 4 * 1024 * 1024;
      public static string StorageConnectionString;
      public static string ServiceBusConnectionString;
   
      private CloudBlobClient blobClient;
      private CloudBlobContainer blobContainer;
      private QueueClient queueClient;
   
      private long currentBlockInitOffset;
      private MemoryStream toAppend = new MemoryStream(MAX_BLOCK_SIZE);
   
      private Stopwatch stopwatch;
      private TimeSpan MAX_CHECKPOINT_TIME = TimeSpan.FromHours(1);
   
      public StoreEventProcessor()
      {
        var storageAccount = CloudStorageAccount.Parse(StorageConnectionString);
        blobClient = storageAccount.CreateCloudBlobClient();
        blobContainer = blobClient.GetContainerReference("d2ctutorial");
        blobContainer.CreateIfNotExists();
        queueClient = QueueClient.CreateFromConnectionString(ServiceBusConnectionString);
      }
   
      Task IEventProcessor.CloseAsync(PartitionContext context, CloseReason reason)
      {
        Console.WriteLine("Processor Shutting Down. Partition '{0}', Reason: '{1}'.", context.Lease.PartitionId, reason);
        return Task.FromResult<object>(null);
      }
   
      Task IEventProcessor.OpenAsync(PartitionContext context)
      {
        Console.WriteLine("StoreEventProcessor initialized.  Partition: '{0}', Offset: '{1}'", context.Lease.PartitionId, context.Lease.Offset);
   
        if (!long.TryParse(context.Lease.Offset, out currentBlockInitOffset))
        {
          currentBlockInitOffset = 0;
        }
        stopwatch = new Stopwatch();
        stopwatch.Start();
   
        return Task.FromResult<object>(null);
      }
   
      async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
      {
        foreach (EventData eventData in messages)
        {
          byte[] data = eventData.GetBytes();
   
          if (eventData.Properties.ContainsKey("messageType") && (string) eventData.Properties["messageType"] == "interactive")
          {
            var messageId = (string) eventData.SystemProperties["message-id"];
   
            var queueMessage = new BrokeredMessage(new MemoryStream(data));
            queueMessage.MessageId = messageId;
            queueMessage.Properties["messageType"] = "interactive";
            await queueClient.SendAsync(queueMessage);
   
            WriteHighlightedMessage(string.Format("Received interactive message: {0}", messageId));
            continue;
          }
   
          if (toAppend.Length + data.Length > MAX_BLOCK_SIZE || stopwatch.Elapsed > MAX_CHECKPOINT_TIME)
          {
            await AppendAndCheckpoint(context);
          }
          await toAppend.WriteAsync(data, 0, data.Length);
   
          Console.WriteLine(string.Format("Message received.  Partition: '{0}', Data: '{1}'",
            context.Lease.PartitionId, Encoding.UTF8.GetString(data)));
        }
      }
   
      private async Task AppendAndCheckpoint(PartitionContext context)
      {
        var blockIdString = String.Format("startSeq:{0}", currentBlockInitOffset.ToString("0000000000000000000000000"));
        var blockId = Convert.ToBase64String(ASCIIEncoding.ASCII.GetBytes(blockIdString));
        toAppend.Seek(0, SeekOrigin.Begin);
        byte[] md5 = MD5.Create().ComputeHash(toAppend);
        toAppend.Seek(0, SeekOrigin.Begin);
   
        var blobName = String.Format("iothubd2c_{0}", context.Lease.PartitionId);
        var currentBlob = blobContainer.GetBlockBlobReference(blobName);
   
        if (await currentBlob.ExistsAsync())
        {
          await currentBlob.PutBlockAsync(blockId, toAppend, Convert.ToBase64String(md5));
          var blockList = await currentBlob.DownloadBlockListAsync();
          var newBlockList = new List<string>(blockList.Select(b => b.Name));
   
          if (newBlockList.Count() > 0 && newBlockList.Last() != blockId)
          {
            newBlockList.Add(blockId);
            WriteHighlightedMessage(String.Format("Appending block id: {0} to blob: {1}", blockIdString, currentBlob.Name));
          }
          else
          {
            WriteHighlightedMessage(String.Format("Overwriting block id: {0}", blockIdString));
          }
          await currentBlob.PutBlockListAsync(newBlockList);
        }
        else
        {
          await currentBlob.PutBlockAsync(blockId, toAppend, Convert.ToBase64String(md5));
          var newBlockList = new List<string>();
          newBlockList.Add(blockId);
          await currentBlob.PutBlockListAsync(newBlockList);
   
          WriteHighlightedMessage(String.Format("Created new blob", currentBlob.Name));
        }
   
        toAppend.Dispose();
        toAppend = new MemoryStream(MAX_BLOCK_SIZE);
   
        // checkpoint.
        await context.CheckpointAsync();
        WriteHighlightedMessage(String.Format("Checkpointed partition: {0}", context.Lease.PartitionId));
   
        currentBlockInitOffset = long.Parse(context.Lease.Offset);
        stopwatch.Restart();
      }
   
      private void WriteHighlightedMessage(string message)
      {
        Console.ForegroundColor = ConsoleColor.Yellow;
        Console.WriteLine(message);
        Console.ResetColor();
      }
    }
    ```
   
    La classe **EventProcessorHost** appelle cette classe pour traiter les messages appareil-à-cloud reçus d’IoT Hub. Le code dans cette classe implémente la logique, afin de stocker les messages de manière fiable dans un conteneur d’objets blob et de transférer les messages interactifs à la file d’attente Service Bus.
   
    La méthode **OpenAsync** initialise la variable **currentBlockInitOffset** qui suit le décalage actuel du premier message lu par ce processeur d’événements. N’oubliez pas : chaque processeur est chargé d’une partition unique.
   
    La méthode **ProcessEventsAsync** reçoit un lot de messages d’IoT Hub et les traite comme suit : elle envoie des messages interactifs à la file d’attente Service Bus et ajoute les messages de point de données à la mémoire tampon appelée **toAppend**. Si la mémoire tampon atteint la limite de 4 Mo ou si la fenêtre de temps de déduplication s’est écoulée (une heure depuis le dernier point de contrôle dans ce didacticiel), un point de contrôle est déclenché par l’application.
   
    La méthode **AppendAndCheckpoint** génère d’abord un ID de bloc pour le bloc à ajouter. Azure Storage requiert que tous les ID de bloc aient la même longueur, la méthode remplit donc le décalage avec des zéros non significatifs - `currentBlockInitOffset.ToString("0000000000000000000000000")`. Ensuite, si un bloc avec cet ID se trouve déjà dans l’objet blob, la méthode le remplace par le contenu actuel de la mémoire tampon.
   
   > [!NOTE]
   > Pour simplifier le code, ce didacticiel utilise un fichier blob unique par partition pour stocker les messages. La solution serait d’implémenter une substitution de fichier, en créant des fichiers supplémentaires lorsqu’ils atteignent une certaine taille ou après un certain laps de temps. N’oubliez pas qu’un objet blob de blocs Azure peut contenir au maximum 195 Go de données.
   > 
   > 
8. Dans la classe **Program**, ajoutez l’instruction **using** suivante en haut :
   
    ```
    using Microsoft.ServiceBus.Messaging;
    ```
9. Modifiez la méthode **Main** dans la classe **Program** comme suit. Remplacez **{iot hub connection string}** par la chaîne de connexion **iothubowner** à partir du didacticiel [Prise en main d’IoT Hub]. Remplacez la chaîne de connexion de stockage par la chaîne de connexion que vous avez notée au début de cette section. Remplacez la chaîne de connexion Service Bus par les autorisations **Send** pour la file d’attente nommée **d2ctutorial** que vous avez notée au début de cette section :
   
    ```
    static void Main(string[] args)
    {
      string iotHubConnectionString = "{iot hub connection string}";
      string iotHubD2cEndpoint = "messages/events";
      StoreEventProcessor.StorageConnectionString = "{storage connection string}";
      StoreEventProcessor.ServiceBusConnectionString = "{service bus send connection string}";
   
      string eventProcessorHostName = Guid.NewGuid().ToString();
      EventProcessorHost eventProcessorHost = new EventProcessorHost(eventProcessorHostName, iotHubD2cEndpoint, EventHubConsumerGroup.DefaultGroupName, iotHubConnectionString, StoreEventProcessor.StorageConnectionString, "messages-events");
      Console.WriteLine("Registering EventProcessor...");
      eventProcessorHost.RegisterEventProcessorAsync<StoreEventProcessor>().Wait();
   
      Console.WriteLine("Receiving. Press enter key to stop worker.");
      Console.ReadLine();
      eventProcessorHost.UnregisterEventProcessorAsync().Wait();
    }
    ```
   
   > [!NOTE]
   > Par souci de simplicité, ce didacticiel utilise une seule instance de la classe [EventProcessorHost] . Pour plus d’informations, consultez le [Guide de programmation de hubs d’événements].
   > 
   > 

## <a name="receive-interactive-messages"></a>Recevoir des messages interactifs
Dans cette section, vous écrivez une application de console Windows qui reçoit les messages interactifs à partir de la file d’attente Service Bus. Pour plus d’informations sur la construction d’une solution à l’aide de Service Bus, voir [Application multiniveau .NET avec les files d’attente Azure Service Bus][].

1. Dans la solution Visual Studio actuelle, créez un projet Windows Visual C# à l’aide du modèle de projet **d’application de console** . Nommez le projet **ProcessD2CInteractiveMessages**.
2. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **ProcessD2CInteractiveMessages**, puis cliquez sur **Gérer les packages NuGet**. Cela affiche la fenêtre **Gestionnaire de packages NuGet**.
3. Recherchez **WindowsAzure.ServiceBus**, cliquez sur **Installer**, puis acceptez les conditions d’utilisation. Cette opération lance le téléchargement, l’installation et ajoute une référence à [Azure Service Bus](https://www.nuget.org/packages/WindowsAzure.ServiceBus)avec toutes ses dépendances.
4. Ajoutez les instructions **using** suivantes au début du fichier **Program.cs** :
   
    ```
    using System.IO;
    using Microsoft.ServiceBus.Messaging;
    ```
5. Enfin, ajoutez les lignes suivantes à la méthode **Main** . Remplacez la chaîne de connexion par des autorisations **Listen** pour la file d’attente nommée **d2ctutorial** :
   
    ```
    Console.WriteLine("Process D2C Interactive Messages app\n");
   
    string connectionString = "{service bus listen connection string}";
    QueueClient Client = QueueClient.CreateFromConnectionString(connectionString);
   
    OnMessageOptions options = new OnMessageOptions();
    options.AutoComplete = false;
    options.AutoRenewTimeout = TimeSpan.FromMinutes(1);
   
    Client.OnMessage((message) =>
    {
      try
      {
        var bodyStream = message.GetBody<Stream>();
        bodyStream.Position = 0;
        var bodyAsString = new StreamReader(bodyStream, Encoding.ASCII).ReadToEnd();
   
        Console.WriteLine("Received message: {0} messageId: {1}", bodyAsString, message.MessageId);
   
        message.Complete();
      }
      catch (Exception)
      {
        message.Abandon();
      }
    }, options);
   
    Console.WriteLine("Receiving interactive messages from SB queue...");
    Console.WriteLine("Press any key to exit.");
    Console.ReadLine();
    ```

## <a name="run-the-applications"></a>Exécution des applications
Vous êtes maintenant prêt à exécuter les applications.

1. Dans Visual Studio, dans l’Explorateur de solutions, cliquez avec le bouton droit sur votre solution, puis sélectionnez **Définir les projets de démarrage**. Sélectionnez **Plusieurs projets de démarrage**, puis sélectionnez l’action **Démarrer** pour les projets **ProcessDeviceToCloudMessages**, **SimulatedDevice** et **ProcessD2CInteractiveMessages**.
2. Appuyez sur **F5** pour lancer les trois applications de console. L’application **ProcessD2CInteractiveMessages** doit traiter chaque message interactif envoyé à partir de l’application **SimulatedDevice**.
   
   ![Trois applications de console][50]

> [!NOTE]
> Pour afficher les mises à jour dans votre fichier blob, vous devrez peut-être définir la constante **MAX_BLOCK_SIZE** dans la classe **StoreEventProcessor** sur une valeur inférieure, telle que **1024**. Cette modification est utile car il faut un certain temps pour atteindre la limite de taille de bloc avec les données envoyées par l’appareil simulé. Avec une plus petite taille de bloc, vous n’avez pas attendre si longtemps avant la création et la mise à jour de l’objet blob. Sachez toutefois qu’une plus grande taille de bloc favorise l’évolutivité de l’application.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à traiter de manière fiable des messages de point de données et des messages interactifs appareil-à-cloud à l’aide de la classe [EventProcessorHost] .

Le didacticiel [Envoi de messages cloud-à-appareil avec IoT Hub][lnk-c2d] montre comment envoyer des messages à vos appareils depuis votre serveur principal.

Pour des exemples de solutions de bout en bout qui utilisent IoT Hub, voir [Azure IoT Suite][lnk-suite].

Pour en savoir plus sur le développement de solutions avec IoT Hub, consultez le [Guide du développeur IoT Hub].

<!-- Images. -->
[50]: ./media/iot-hub-csharp-csharp-process-d2c/run1.png
[10]: ./media/iot-hub-csharp-csharp-process-d2c/create-identity-csharp1.png

[30]: ./media/iot-hub-csharp-csharp-process-d2c/createqueue2.png
[31]: ./media/iot-hub-csharp-csharp-process-d2c/createqueue3.png
[32]: ./media/iot-hub-csharp-csharp-process-d2c/createqueue4.png

<!-- Links -->

[stockage d’objets blob Azure]: ../storage/storage-dotnet-how-to-use-blobs.md
[Azure Data Factory]: https://azure.microsoft.com/documentation/services/data-factory/
[HDInsight (Hadoop)]: https://azure.microsoft.com/documentation/services/hdinsight/
[file d’attente Azure Service Bus]: ../service-bus-messaging/service-bus-dotnet-get-started-with-queues.md

[Guide du développeur Azure IoT Hub - appareil-à-cloud]: iot-hub-devguide-messaging.md

[Stockage Azure]: https://azure.microsoft.com/documentation/services/storage/
[Azure Service Bus]: https://azure.microsoft.com/documentation/services/service-bus/

[Guide du développeur IoT Hub]: iot-hub-devguide.md
[Prise en main d’IoT Hub]: iot-hub-csharp-csharp-getstarted.md
[Centre de développement Azure IoT]: https://azure.microsoft.com/develop/iot
[lnk-service-fabric]: https://azure.microsoft.com/documentation/services/service-fabric/
[lnk-stream-analytics]: https://azure.microsoft.com/documentation/services/stream-analytics/
[lnk-event-hubs]: https://azure.microsoft.com/documentation/services/event-hubs/
[Gestion des erreurs temporaires]: https://msdn.microsoft.com/library/hh675232.aspx

<!-- Links -->
[À propos des comptes de stockage Azure]: ../storage/storage-create-storage-account.md#create-a-storage-account
[Prise en main des hubs d’événements]: ../event-hubs/event-hubs-csharp-ephcs-getstarted.md
[Objectifs de performance et évolutivité d’Azure Storage]: ../storage/storage-scalability-targets.md
[Objets blob de blocs Azure]: https://msdn.microsoft.com/library/azure/ee691964.aspx
[Event Hubs]: ../event-hubs/event-hubs-overview.md
[EventProcessorHost]: http://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.eventprocessorhost(v=azure.95).aspx
[Guide de programmation de concentrateurs d’événements]: ../event-hubs/event-hubs-programming-guide.md
[Transient Fault Handling]: https://msdn.microsoft.com/library/hh680901(v=pandp.50).aspx
[Créer des applications multiniveaux avec Service Bus]: ../service-bus-messaging/service-bus-dotnet-multi-tier-app-using-service-bus-queues.md

[portail classique d’un lien]: https://manage.windowsazure.com
[lnk-c2d]: iot-hub-csharp-csharp-process-d2c.md
[lnk-suite]: https://azure.microsoft.com/documentation/suites/iot-suite/



<!----HONumber=Oct16_HO2-->


