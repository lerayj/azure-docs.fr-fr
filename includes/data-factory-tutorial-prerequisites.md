Dans ce didacticiel, vous devez générer votre première fabrique de données Azure avec un pipeline de données qui traite les données en exécutant le script Hive sur un cluster Azure HDInsight (Hadoop).

Cet article ne fournit pas de vue d’ensemble conceptuelle d’Azure Data Factory. Pour obtenir une vue d’ensemble conceptuelle de ce service, consultez [Présentation d’Azure Data Factory](../articles/data-factory/data-factory-introduction.md).

## Sujets traités dans ce didacticiel
**Azure Data Factory** permet de concevoir des tâches de **déplacement** et de **traitement** de données dans des flux de travail pilotés par les données (également appelés pipelines de données). Vous apprenez à créer votre premier pipeline de données avec une tâche de traitement de données (ou de transformation de données). Cette tâche utilise un cluster Azure HDInsight pour transformer et analyser les journaux web.

Dans ce didacticiel, vous effectuerez les étapes suivantes :

1. Créer la **fabrique de données**. Une fabrique de données peut contenir un ou plusieurs pipelines de données qui déplacent et traitent des données.
2. Créer le **service lié**. Vous créez un service lié pour lier un magasin de données ou un service de calcul à la fabrique de données. Un magasin de données comme Azure Storage conserve les données d’entrée/de sortie d’activités dans le pipeline. Un service de calcul comme Azure HDInsight traite/transforme des données.
3. Créer des **jeux de données** d’entrée et de sortie. Un jeu de données d’entrée représente l’entrée d’une activité dans le pipeline, tandis qu’un jeu de données de sortie représente la sortie de l’activité.
4. Créer le **pipeline**. Un pipeline peut avoir une ou plusieurs activités (exemples : activité de copie, activité Hive HDInsight). Cet exemple utilise l’activité Hive d’HDInsight, qui exécute un script Hive. Le script crée d’abord une table externe qui fait référence aux données de journaux web bruts stockées dans le stockage d’objets blob Azure, puis partitionne les données brutes par année et par mois.

![Vue Diagramme dans le didacticiel Data Factory](./media/data-factory-tutorial-prerequisites/data-factory-tutorial-diagram-view.png)

Dans ce didacticiel, adfgetstarted (conteneur) => inputdata (dossier) contient un fichier nommé input.log. Ce fichier journal contient des entrées de trois mois : janvier, février et mars 2014. Voici les échantillons de lignes pour chaque mois du fichier d’entrée.

    2014-01-01,02:01:09,SAMPLEWEBSITE,GET,/blogposts/mvc4/step2.png,X-ARR-LOG-ID=2ec4b8ad-3cf0-4442-93ab-837317ece6a1,80,-,1.54.23.196,Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36,-,http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx,\N,200,0,0,53175,871 
    2014-02-01,02:01:10,SAMPLEWEBSITE,GET,/blogposts/mvc4/step7.png,X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c,80,-,1.54.23.196,Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36,-,http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx,\N,200,0,0,30184,871
    2014-03-01,02:01:10,SAMPLEWEBSITE,GET,/blogposts/mvc4/step7.png,X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c,80,-,1.54.23.196,Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36,-,http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx,\N,200,0,0,30184,871

Quand le fichier est traité par le pipeline avec l’activité Hive d’HDInsight, l’activité exécute un script Hive sur le cluster HDInsight qui partitionne les données d’entrée par année et par mois. Le script crée trois dossiers de sortie qui contiennent un fichier avec les entrées de chaque mois.

    adfgetstarted/partitioneddata/year=2014/month=1/000000_0
    adfgetstarted/partitioneddata/year=2014/month=2/000000_0
    adfgetstarted/partitioneddata/year=2014/month=3/000000_0

Dans les échantillons de lignes ci-dessus, la première (avec 2014-01-01) est écrite dans le fichier 000000\_0 dans le dossier month=1. De même, la deuxième est écrite dans le fichier du dossier month=2 et la troisième est écrite dans le fichier du dossier month=3.

## Conditions préalables
Avant de commencer ce didacticiel, vous devez disposer des éléments suivants :

1. **Un abonnement Azure** : si vous n’en avez pas, vous pouvez créer un compte en quelques minutes pour une évaluation gratuite. Consultez l’article [Évaluation gratuite](https://azure.microsoft.com/pricing/free-trial/) pour savoir comment obtenir un compte d’évaluation gratuite.
2. **Stockage Azure** : dans ce didacticiel, vous utilisez un compte de stockage Azure pour stocker les données. Si vous n’avez pas de compte de stockage Azure, consultez l’article [Créer un compte de stockage](../articles/storage/storage-create-storage-account.md#create-a-storage-account). Après avoir créé le compte de stockage, vous devez obtenir la clé du compte utilisée pour accéder au stockage. Consultez [Affichage, copie et régénération de clés d’accès de stockage](../articles/storage/storage-create-storage-account.md#view-and-copy-storage-access-keys).

## Charger des fichiers dans Azure Storage pour le didacticiel
En premier lieu, vous devez préparer le compte de stockage Azure et ses fichiers nécessaires pour commencer ce didacticiel.

Dans cette section, vous effectuerez les tâches suivantes :

1. Charger le fichier de requête Hive (HQL) dans le dossier **script** du conteneur **adfgetstarted**.
2. Charger le fichier d’entrée dans le dossier **inputdata** du conteneur **adfgetstarted**.

### Créer le fichier de script HQL
1. Lancez le **Bloc-notes** et collez le script HQL suivant. Ce script Hive crée deux tables : **WebLogsRaw** et **WebLogsPartitioned**. Cliquez sur **Fichier** dans le menu et sélectionnez **Enregistrer sous**. Accédez au dossier **C:\\adfgetstarted** de votre disque dur. Sélectionnez **Tous les fichiers (*.*)** pour le champ **Type de fichier**. Entrez **partitionweblogs.hql** pour le **Nom de fichier**. Vérifiez que le champ **Codage** au bas de la boîte de dialogue est défini sur **ANSI**. Si ce n’est pas le cas, définissez-le sur **ANSI**.
   
        set hive.exec.dynamic.partition.mode=nonstrict;
   
        DROP TABLE IF EXISTS WebLogsRaw; 
        CREATE TABLE WebLogsRaw (
          date  date,
          time  string,
          ssitename string,
          csmethod  string,
          csuristem  string,
          csuriquery string,
          sport int,
          susername string,
          cipcsUserAgent string,
          csCookie string,
          csReferer string,
          cshost  string,
          scstatus  int,
          scsubstatus  int,
          scwin32status  int,
          scbytes int,
          csbytes int,
          timetaken int
        )
        ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
        LINES TERMINATED BY '\n' 
        tblproperties ("skip.header.line.count"="2");
   
        LOAD DATA INPATH '${hiveconf:inputtable}' OVERWRITE INTO TABLE WebLogsRaw;
   
        DROP TABLE IF EXISTS WebLogsPartitioned ; 
        create external table WebLogsPartitioned (  
          date  date,
          time  string,
          ssitename string,
          csmethod  string,
          csuristem  string,
          csuriquery string,
          sport int,
          susername string,
          cipcsUserAgent string,
          csCookie string,
          csReferer string,
          cshost  string,
          scstatus  int,
          scsubstatus  int,
          scwin32status  int,
          scbytes int,
          csbytes int,
          timetaken int
        )
        partitioned by ( year int, month int)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' 
        STORED AS TEXTFILE 
        LOCATION '${hiveconf:partitionedtable}';
   
        INSERT INTO TABLE WebLogsPartitioned  PARTITION( year , month) 
        SELECT
          date,
          time,
          ssitename,
          csmethod,
          csuristem,
          csuriquery,
          sport,
          susername,
          cipcsUserAgent,
          csCookie,
          csReferer,
          cshost,
          scstatus,
          scsubstatus,
          scwin32status,
          scbytes,
          csbytes,
          timetaken,
          year(date),
          month(date)
        FROM WebLogsRaw

Lors de l’exécution, l’activité Hive dans le pipeline Data Factory transmet les valeurs pour les paramètres inputtable et partitionedtable comme indiqué dans l’extrait de code suivant. La valeur storageaccountname est le nom de votre compte de stockage Azure.

        "inputtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/inputdata",
        "partitionedtable": "wasb://adfgetstarted@<storageaccountname>.blob.core.windows.net/partitioneddata"

### Créer un exemple de fichier d’entrée
Dans le Bloc-notes, créez un fichier nommé **input.log** dans le dossier **c:\\adfgetstarted** avec le contenu suivant :

    #Software: Microsoft Internet Information Services 8.0
    #Fields: date time s-sitename cs-method cs-uri-stem cs-uri-query s-port cs-username c-ip cs(User-Agent) cs(Cookie) cs(Referer) cs-host sc-status sc-substatus sc-win32-status sc-bytes cs-bytes time-taken
    2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step2.png X-ARR-LOG-ID=2ec4b8ad-3cf0-4442-93ab-837317ece6a1 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 53175 871 46
    2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step3.png X-ARR-LOG-ID=9eace870-2f49-4efd-b204-0d170da46b4a 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 51237 871 32
    2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step4.png X-ARR-LOG-ID=4bea5b3d-8ac9-46c9-9b8c-ec3e9500cbea 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 72177 871 47
    2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step5.png X-ARR-LOG-ID=9b0c14b1-434d-495a-9b0d-46775194257b 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 37931 871 32
    2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step6.png X-ARR-LOG-ID=f99cff81-ec38-4a13-b2fe-21b10adca996 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 27146 871 47
    2014-01-01 02:01:09 SAMPLEWEBSITE GET /blogposts/mvc4/step1.png X-ARR-LOG-ID=af94d3b4-8e05-4871-82c4-638f866d4e83 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 66259 871 140
    2014-02-01 02:01:10 SAMPLEWEBSITE GET /blogposts/mvc4/step7.png X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 30184 871 47
    2014-02-01 02:01:10 SAMPLEWEBSITE GET /blogposts/mvc4/step7.png X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 30184 871 47
    2014-02-01 02:01:10 SAMPLEWEBSITE GET /blogposts/mvc4/step7.png X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 30184 871 47
    2014-02-01 02:01:10 SAMPLEWEBSITE GET /blogposts/mvc4/step7.png X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 30184 871 47
    2014-02-01 02:01:10 SAMPLEWEBSITE GET /blogposts/mvc4/step7.png X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 30184 871 47
    2014-02-01 02:01:10 SAMPLEWEBSITE GET /blogposts/mvc4/step7.png X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 30184 871 47
    2014-02-01 02:01:10 SAMPLEWEBSITE GET /blogposts/mvc4/step7.png X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 30184 871 47
    2014-02-01 02:01:10 SAMPLEWEBSITE GET /blogposts/mvc4/step7.png X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 30184 871 47
    2014-03-01 02:01:10 SAMPLEWEBSITE GET /blogposts/mvc4/step7.png X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 30184 871 47
    2014-03-01 02:01:10 SAMPLEWEBSITE GET /blogposts/mvc4/step7.png X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 30184 871 47
    2014-03-01 02:01:10 SAMPLEWEBSITE GET /blogposts/mvc4/step7.png X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 30184 871 47
    2014-03-01 02:01:10 SAMPLEWEBSITE GET /blogposts/mvc4/step7.png X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 30184 871 47
    2014-03-01 02:01:10 SAMPLEWEBSITE GET /blogposts/mvc4/step7.png X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c 80 - 1.54.23.196 Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36 - http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx www.sample.com 200 0 0 30184 871 47

### Charger le fichier d’entrée et le fichier HQL dans votre stockage d’objets blob Azure
Cette section explique comment utiliser l’outil **AzCopy** pour copier des fichiers dans le stockage d’objets Blob Azure. Vous pouvez utiliser n’importe quel outil de votre choix (par exemple [l’Explorateur de stockage Microsoft Azure](http://storageexplorer.com/), [CloudXPlorer de ClumsyLeaf Software](http://clumsyleaf.com/products/cloudxplorer)) pour effectuer cette tâche.

1. Pour préparer le stockage Azure en vue du didacticiel, procédez comme suit :
   
   1. Téléchargez la [dernière version d’**AzCopy**](http://aka.ms/downloadazcopy) ou la [dernière version Preview](http://aka.ms/downloadazcopypr). Consultez l’article [Utilisation d’AzCopy](../articles/storage/storage-use-azcopy.md) pour obtenir des instructions sur l’utilisation de l’utilitaire.
   2. Ajoutez l’emplacement AzCopy au chemin système en exécutant la commande suivante dans l’invite de commandes.
      
           set path=%path%;C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy
   3. Accédez au dossier c:\\adfgetstarted et exécutez la commande suivante. Cette commande permet de télécharger le fichier **input.log** dans le compte de stockage (conteneur **adfgetstarted** et dossier **inputdata**). Remplacez **StorageAccountName** par le nom de votre compte de stockage et **Storage Key** par la clé du compte de stockage.
      
           AzCopy /Source:. /Dest:https://<storageaccountname>.blob.core.windows.net/adfgetstarted/inputdata /DestKey:<storagekey>  /Pattern:input.log
      
      > [!NOTE]
      > Cette commande crée un conteneur nommé **adfgetstarted** dans votre stockage d’objets blob Azure et copie le fichier **input.log** de votre disque local vers le dossier **inputdata** du conteneur.
      > 
      > 
   4. Une fois que le fichier a été chargé, vous voyez une sortie d’AzCopy similaire à la suivante.
      
           Finished 1 of total 1 file(s).
           [2015/12/16 23:07:33] Transfer summary:
           -----------------
           Total files transferred: 1
           Transfer successfully:   1
           Transfer skipped:        0
           Transfer failed:         0
           Elapsed time:            00.00:00:01
   5. Exécutez la commande suivante pour charger le fichier **partitionweblogs.hql** dans le dossier **script** du conteneur **adfgetstarted**. Voici la commande :
      
           AzCopy /Source:. /Dest:https://<storageaccountname>.blob.core.windows.net/adfgetstarted/script /DestKey:<storagekey>  /Pattern:partitionweblogs.hql

<!---HONumber=AcomDC_0914_2016-->