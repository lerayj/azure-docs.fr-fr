---
title: "Gérer des profils Azure Traffic Manager | Microsoft Docs"
description: "Cet article vous aide à créer, désactiver, activer, supprimer et afficher l’historique d’un profil Azure Traffic Manager."
services: traffic-manager
documentationcenter: 
author: sdwheeler
manager: carmonm
editor: 
ms.assetid: f06e0365-0a20-4d08-b7e1-e56025e64f66
ms.service: traffic-manager
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/11/2016
ms.author: sewhee
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 7c322fef4fb8588d6d3cc5c6c30fdcb486834155

---

# <a name="manage-an-azure-traffic-manager-profile"></a>Gestion d’un profil Azure Traffic Manager

Les profils Traffic Manager permettent de contrôler la distribution du trafic vers les points de terminaison de votre site Web ou de vos services cloud selon des méthodes de routage du trafic. Cet article explique comment créer et gérer ces profils.

## <a name="create-a-traffic-manager-profile-using-quick-create"></a>Création d’un profil Traffic Manager avec l’option Création rapide

Vous pouvez créer un profil Traffic Manager rapidement à l’aide de l’option Création rapide dans le portail Azure Classic. Cette option vous permet de créer des profils dotés de paramètres de configuration de base. Toutefois, vous ne pouvez pas utiliser la Création rapide pour définir des paramètres tels que l’ensemble de points de terminaison (services cloud et sites web), l’ordre de basculement pour la méthode de routage du trafic de basculement ou la surveillance. Après avoir créé votre profil, vous pouvez configurer ces paramètres dans le portail Azure Classic. Traffic Manager prend en charge jusqu’à 200 points de terminaison par profil. Toutefois, la plupart des scénarios d’utilisation ne requièrent que peu de points de terminaison.

### <a name="to-create-a-traffic-manager-profile"></a>Pour créer un profil Traffic Manager

1. **Déployez vos services cloud et sites web vers votre environnement de production.** Pour en savoir plus sur les services cloud, consultez [Cloud Services](http://go.microsoft.com/fwlink/p/?LinkId=314074). Pour en savoir plus sur les sites web, consultez [Sites web](http://go.microsoft.com/fwlink/p/?LinkId=393327).
2. **Connectez-vous au Portail Azure Classic.** Cliquez sur **Nouveau** dans l’angle inférieur gauche du portail, puis cliquez sur **Network Services > Traffic Manager**, puis **Création rapide** pour commencer à configurer votre profil.
3. **Configurez le préfixe DNS.**  Donnez un nom de préfixe DNS unique à votre profil Traffic Manager. Vous pouvez spécifier uniquement le préfixe d’un nom de domaine Traffic Manager.
4. **Sélectionnez l’abonnement.**  Sélectionnez l’abonnement Azure approprié. Chaque profil est associé à un seul abonnement. Si vous n’avez qu’un abonnement, cette option n’apparaît pas.
5. **Sélectionnez la méthode de routage du trafic.** Sélectionnez la méthode de routage du trafic dans la **stratégie de routage du trafic**. Pour plus d’informations sur les différentes méthodes de routage du trafic, consultez [À propos des méthodes de routage du trafic de Traffic Manager](traffic-manager-routing-methods.md).
6. **Cliquez sur « Créer » pour créer le profil**. Lorsque la configuration du profil est terminée, recherchez-le dans le volet Traffic Manager du portail Azure Classic.
7. **Configurez les points de terminaison, la surveillance et les paramètres supplémentaires dans le portail Azure Classic.** La création rapide configure uniquement les paramètres de base. Il est nécessaire de configurer des paramètres supplémentaires comme la liste des points de terminaison et l’ordre de basculement des points de terminaison.

## <a name="disable-enable-or-delete-a-profile"></a>Désactiver, activer ou supprimer un profil

Vous pouvez désactiver un profil existant afin que Traffic Manager ne renvoie pas les demandes utilisateur vers les points de terminaison configurés. Lorsque vous désactivez un profil Traffic Manager, le profil et les informations qu’il contient demeurent intacts et peuvent être modifiés via l’interface de Traffic Manager.  Les références sont rétablies lorsque vous réactivez le profil. Lorsque vous créez un profil Traffic Manager dans le Portail Azure Classic, il est automatiquement activé. Si un profil ne vous semble plus nécessaire, vous pouvez le supprimer.

### <a name="to-disable-a-profile"></a>Désactivation d’un profil

1. Si vous utilisez un nom de domaine personnalisé, modifiez l’enregistrement CNAME sur votre serveur DNS Internet afin qu’il ne pointe plus sur votre profil Traffic Manager.
2. Le trafic n’est alors plus dirigé vers les points de terminaison via les paramètres du profil Traffic Manager.
3. Sélectionnez le profil que vous souhaitez désactiver. Mettez le profil en surbrillance sur la page Traffic Manager en cliquant sur la colonne en regard du nom de profil. Remarque : si vous cliquez sur le nom du profil ou sur la flèche en regard du nom, la page de paramètres du profil s’ouvre.
4. Après avoir sélectionné le profil, cliquez sur **Désactiver** en bas de la page.

### <a name="to-enable-a-profile"></a>Activation d’un profil

1. Sélectionnez le profil que vous souhaitez désactiver. Mettez le profil en surbrillance sur la page Traffic Manager en cliquant sur la colonne en regard du nom de profil. Remarque : si vous cliquez sur le nom du profil ou sur la flèche en regard du nom, la page de paramètres du profil s’ouvre.
2. Après avoir sélectionné le profil, cliquez sur **Activer** en bas de la page.
3. Si vous utilisez un nom de domaine personnalisé, créez un enregistrement de ressource CNAME sur votre serveur DNS Internet qui pointe sur le nom de domaine de votre profil Traffic Manager.
4. Le trafic est de nouveau dirigé vers les points de terminaison.

### <a name="to-delete-a-profile"></a>Suppression d’un profil

1. Assurez-vous que l’enregistrement de ressource DNS sur votre serveur DNS Internet n’utilise plus un enregistrement de ressource CNAME pointant vers le nom de domaine de votre profil Traffic Manager.
2. Sélectionnez le profil que vous souhaitez désactiver. Mettez le profil en surbrillance sur la page Traffic Manager en cliquant sur la colonne en regard du nom de profil. Remarque : si vous cliquez sur le nom du profil ou sur la flèche en regard du nom, la page de paramètres du profil s’ouvre.
3. Après avoir sélectionné le profil, cliquez sur **Supprimer** en bas de la page.

## <a name="view-traffic-manager-profile-change-history"></a>Afficher l'historique des modifications d'un profil Traffic Manager

Vous pouvez afficher l’historique des modifications de votre profil Traffic Manager dans les services de gestion du portail Azure Classic.

### <a name="to-view-your-traffic-manager-change-history"></a>Pour afficher votre historique des modifications Traffic Manager

1. Dans le volet gauche du portail Azure Classic, cliquez sur **Services de gestion**.
2. Dans la page Services de gestion, cliquez sur **Journaux des opérations**.
3. Dans la page Journaux des opérations, vous pouvez filtrer pour afficher l’historique des modifications de votre profil Traffic Manager. Après avoir sélectionné les options de filtrage, cliquez sur la coche pour afficher les résultats.

   * Pour afficher les modifications apportées à tous vos profils, sélectionnez votre abonnement et l’intervalle de temps, puis **Traffic Manager** dans le menu contextuel **Type**.
   * Pour filtrer par nom de profil, tapez le nom du profil dans le champ **Nom du service** ou sélectionnez-le dans le menu contextuel.
   * Pour afficher les détails de chaque modification, sélectionnez la ligne contenant la modification à afficher, puis cliquez sur **Détails** en bas de la page. Dans la fenêtre **Détails de l’opération** , vous pouvez afficher la représentation XML de l’objet API qui a été créé ou mis à jour dans le cadre de l’opération.

## <a name="next-steps"></a>Étapes suivantes

* [Ajout d’un point de terminaison](traffic-manager-endpoints.md)
* [Configurer la méthode de routage par basculement](traffic-manager-configure-failover-routing-method.md)
* [Configurer la méthode de routage du trafic en tourniquet (round robin)](traffic-manager-configure-round-robin-routing-method.md)
* [Configurer la méthode de routage basé sur les performances](traffic-manager-configure-performance-routing-method.md)
* [Rediriger un domaine Internet d’entreprise vers un nom de domaine Traffic Manager](traffic-manager-point-internet-domain.md)
* [Résolution des problèmes liés à l’état Détérioré de Traffic Manager](traffic-manager-troubleshooting-degraded.md)



<!--HONumber=Nov16_HO2-->


