---
title: "Présentation détaillée des plans Azure App Service | Microsoft Docs"
description: "Découvrez comment fonctionnent les plans Azure App Service et comment ils peuvent améliorer votre gestion."
keywords: "app service, azure app service, mise à l&quot;échelle, évolutif, plan app service, coût d&quot;app service"
services: app-service
documentationcenter: 
author: btardif
manager: wpickett
editor: 
ms.assetid: dea3f41e-cf35-481b-a6bc-33d7fc9d01b1
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/13/2016
ms.author: byvinyal
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: 5f4e2775a71c743c313831ce0cd567527c8ae5e2


---
# <a name="azure-app-service-plans-in-depth-overview"></a>Présentation détaillée des plans Azure App Service
Les plans App Service représentent un ensemble de fonctionnalités et de capacités que vous pouvez partager entre différentes applications. Les Web Apps, les Mobile Apps, les Function Apps et les API Apps, dans [Azure App Service](http://go.microsoft.com/fwlink/?LinkId=529714), fonctionnent toutes dans un plan App Service. Ces plans prennent en charge cinq niveaux de tarification : *Gratuit*, *Partagé*, *De base*, *Standard* et *Premium*. Chaque niveau a ses propres capacité et limites. Les applications comprises dans un même abonnement et situées au même emplacement peuvent partager un plan. Toutes les applications qui partagent un plan peuvent tirer profit de l’ensemble des fonctionnalités définies par ce niveau de plan. Toutes les applications web associées à un plan donné utilisent les ressources définies par ce plan.

Par exemple, si votre plan est configuré pour utiliser deux « petites » instances d’un niveau de service standard, toutes les applications associées à ce plan sont exécutées sur ces deux instances et ont accès aux fonctionnalités du niveau de service standard. Les instances de plans sur lesquelles sont exécutées des applications sont entièrement gérées et hautement disponibles.

Cet article présente les principales caractéristiques telles que le niveau et l’échelle d’un plan App Service, et comment elles entrent en jeu lors de la gestion de vos applications.

## <a name="apps-and-app-service-plans"></a>Applications et plans App Service
Une application App Service ne peut être associée qu'à un seul plan App Service à la fois.

Les applications et les plans sont contenus dans un groupe de ressources. Un groupe de ressources sert de limite de cycle de vie pour chaque ressource qu’il contient. Vous pouvez utiliser des groupes de ressources pour gérer toutes les parties d’une application en même temps.

Étant donné qu’un seul groupe de ressources peut disposer de plusieurs plans App Service, vous pouvez allouer différentes applications à différentes ressources physiques. Par exemple, vous pouvez répartir les ressources entre les environnements de développement, de test et de production. Avoir des environnements de production et de développement/test distincts permet d’isoler des ressources. De cette façon, les tests de charge d’une nouvelle version de vos applications ne font pas appel aux mêmes ressources que les applications de production qui sont distribuées aux clients réels.

Lorsque vous avez plusieurs plans au sein d’un même groupe de ressources, vous pouvez également définir une application disponible pour plusieurs régions géographiques. Par exemple, une application hautement disponible qui s’exécute dans deux régions inclut au moins deux plans, un pour chaque région, et une application associée à chaque plan. Dans ce cas, toutes les copies de l’application appartiennent à un seul groupe de ressources. Le fait de disposer d’un groupe de ressources avec plusieurs plans et plusieurs applications facilite la gestion, le contrôle et l’affichage de l’intégrité de l’application.

## <a name="create-an-app-service-plan-or-use-existing-one"></a>Créer un plan App Service ou utiliser un plan existant
Lorsque vous créez une application, vous devez penser à créer un groupe de ressources. En revanche, si l’application que vous allez créer est un composant d’une application plus importante, elle doit être créée au sein du groupe de ressources alloué pour l’application en question.

Que la nouvelle application soit autonome ou fasse partie d’une autre application, vous pouvez choisir d’utiliser un plan App Service existant pour l’héberger ou bien en créer un. Cette décision relève plus d’une question de capacité et de charge attendue.

Si la nouvelle application est destinée à consommer beaucoup de ressources et à avoir des facteurs d’échelle différents des autres applications hébergées dans un plan existant, il est recommandé de l’isoler dans son propre plan.

Lorsque vous créez un plan, vous pouvez allouer un nouvel ensemble de ressources à votre application et mieux contrôler l’allocation de ressources, car chaque plan obtient son propre ensemble d’instances.

La capacité à déplacer des applications d’un plan à un autre permet également de modifier la façon dont les ressources sont allouées dans toute l’application principale.

Si vous souhaitez créer une application dans une autre région et que celle-ci n’a pas de plan existant, créez un plan dans cette région pour pouvoir y héberger votre application.

## <a name="create-an-app-service-plan"></a>Créer un plan App Service
> [!TIP]
> Si vous disposez d’un environnement App Service, vous pouvez consulter la documentation spécifique aux environnements App Service ici : [Créer un plan App Service dans un environnement App Service](../app-service-web/app-service-web-how-to-create-a-web-app-in-an-ase.md#createplan)
> 
> 

Vous pouvez créer un plan App Service vide à partir de l’expérience de navigation du plan App Service ou dans le cadre de la création d’applications.

Dans le [portail Azure](https://portal.azure.com), cliquez sur **Nouveau** > **Web et mobilité**, puis sélectionnez **Application web** ou tout autre type d’application App Service.
![Créer une application dans le portail Azure.][createWebApp]

Vous pouvez ensuite sélectionner ou créer le plan App Service pour la nouvelle application.

 ![Créer un plan App Service.][createASP]

Pour créer un nouveau plan App Service, cliquez sur **[+] Créer nouveau**, saisissez le nom du **plan App Service**, puis sélectionnez un **emplacement**. Cliquez sur **Niveau de tarification**, puis sélectionnez un niveau de tarification approprié pour le service. Sélectionnez **Afficher tout** pour afficher davantage d’options de tarification, telles que **Gratuit** et **Partagé**. Une fois que vous avez sélectionné le niveau de tarification, cliquez sur le bouton **Sélectionner** .

## <a name="move-an-app-to-a-different-app-service-plan"></a>Déplacer une application vers un autre plan App Service
Vous pouvez déplacer une application vers un autre plan App Service depuis le [portail Azure](https://portal.azure.com). Vous pouvez déplacer les applications d’un plan à l’autre tant que les plans se trouvent dans le même groupe de ressources et dans la même région géographique.

Pour déplacer une application vers un autre plan, accédez à l’application que vous souhaitez déplacer. Dans le menu **Paramètres**, recherchez **Modifier le plan App Service**.

**Modifier le Plan App Service** ouvre le sélecteur de **plan App Service**. À ce stade, vous pouvez choisir un plan existant ou en créer un. Seuls les plans valides (dans le même groupe de ressources et le même emplacement géographique) s’affichent.

![Sélecteur de plan App Service.][change]

Chaque plan a son propre niveau de tarification. Par exemple, quand vous faites passer un site du niveau Gratuit au niveau Standard, votre application peut alors utiliser toutes les fonctionnalités et ressources du niveau Standard.

## <a name="clone-an-app-to-a-different-app-service-plan"></a>Cloner une application vers un autre plan App Service
Si vous souhaitez déplacer l'application vers une autre région, vous pouvez également utiliser le clonage d’application. Le clonage crée une copie de votre application dans un plan ou un environnement App Service, nouveau ou existant, dans n’importe quelle région.

 ![Cloner une application.][appclone]

Vous pouvez trouver **Cloner une application** dans le menu **Outils**.

Pour plus d’informations sur les limites du clonage, voir [Clonage de l’application Azure App Service à l’aide du portail Azure](../app-service-web/app-service-web-app-cloning-portal.md).

## <a name="scale-an-app-service-plan"></a>Mettre à l’échelle un plan App Service
Il existe trois façons de mettre à l'échelle un plan :

* **Modifier le niveau de tarification du plan**. Par exemple, un plan associé au niveau de tarification De base peut être converti en un plan de niveau Standard ou Premium. Toutes les applications associées à ce plan peuvent alors utiliser les fonctionnalités proposées par le nouveau niveau de service.
* **Modifier la taille des instances du plan**. Par exemple, un plan associé au niveau de tarification De base et utilisant des petites instances peut être modifié pour utiliser de grandes instances. Toutes les applications associées à ce plan peuvent dans ce cas utiliser la mémoire supplémentaire et des ressources processeur offertes par l’instance la plus grande.
* **Modifier le nombre d’instances du plan**. Par exemple, un plan Standard réparti sur trois instances peut évoluer vers 10 instances. Un plan Premium peut comporter jusqu’à 20 instances (selon la disponibilité). Toutes les applications associées à ce plan peuvent dans ce cas utiliser la mémoire supplémentaire et des ressources processeur offertes par le plus grand nombre d’instances.

Vous pouvez modifier le niveau tarifaire et la taille de l’instance en cliquant sur l’élément **Montée en puissance** sous les paramètres du plan de l’application ou du plan App Service. Les modifications s’appliquent au plan App Service et affectent toutes les applications hébergées par celui-ci.

 ![Définir des valeurs pour la montée en puissance d’une application.][pricingtier]

## <a name="app-service-plan-cleanup"></a>Nettoyage du plan App Service
Les **plans App Service** qui ne sont associés à aucune application impliquent tout de même des frais, car ils continuent à réserver la capacité de calcul configurée dans les propriétés de mise à l’échelle du plan App Service.
Pour éviter des frais inattendus, lorsque la dernière application hébergée dans un plan App Service est supprimée, le plan App Service vide qui en résulte est également supprimé.

## <a name="summary"></a>Résumé
Les plans App Service représentent un ensemble de fonctionnalités et de capacités que vous pouvez partager entre vos différentes applications. Les plans App Service vous donnent la possibilité d’allouer des applications spécifiques à un ensemble de ressources, et d’optimiser davantage l’utilisation des ressources Azure. Ainsi, si vous souhaitez faire des économies sur votre environnement de test, vous pouvez partager un même plan entre plusieurs applications. Vous pouvez également augmenter le débit de votre environnement de production en le mettant à l'échelle dans plusieurs régions et plusieurs plans.

## <a name="whats-changed"></a>Changements apportés
* Pour obtenir un guide présentant les modifications apportées dans le cadre de la transition entre Sites Web et App Service, consultez la page : [Azure App Service et les services Azure existants](http://go.microsoft.com/fwlink/?LinkId=529714)

[pricingtier]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/appserviceplan-pricingtier.png
[assign]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/assing-appserviceplan.png
[change]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/change-appserviceplan.png
[createASP]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/create-appserviceplan.png
[createWebApp]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/create-web-app.png
[appclone]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/app-clone.png



<!--HONumber=Nov16_HO3-->


