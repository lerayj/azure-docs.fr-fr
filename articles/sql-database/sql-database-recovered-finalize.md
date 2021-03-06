---
title: Finaliser la base de données SQL Microsoft Azure restaurée
description: Limite de restauration dans le temps, base de données SQL Microsoft Azure, restaurer une base de données, récupérer une base de données, portail Azure Classic, portail Azure Classic
services: sql-database
documentationcenter: ''
author: CarlRabeler
manager: jhubbard
editor: ''

ms.service: sql-database
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: storage-backup-recovery
ms.date: 07/05/2016
ms.author: carlrab

---
# Finaliser la base de données SQL Microsoft Azure restaurée
## Vue d'ensemble
Cet article fournit une liste de contrôle des tâches que vous devez effectuer avant de remettre une base de données SQL Azure récemment restaurée en production. Cette liste s’applique aux bases de données restaurées suite à un basculement de géo-réplication, à la restauration d’une base de données supprimée, à une restauration à un moment donné ou à une géo-restauration.

## Mettre à jour les chaînes de connexion
Vérifiez que les chaînes de connexion de votre application pointent bien vers la base de données qui vient d’être restaurée. Mettez vos chaînes de connexion à jour dans l’un des cas suivants :

* La base de données restaurée utilise un nom différent de celui de la base de données source.
* La base de données restaurée se trouve sur un serveur différent du serveur source.

Pour en savoir plus sur la modification des chaînes de connexion, voir les [instructions pour la connexion à la base de données SQL Azure par programme](https://msdn.microsoft.com/library/azure/ee336282.aspx) et [Vue d’ensemble du développement de base de données SQL](sql-database-develop-overview.md).

## Modifier les règles de pare-feu
Vérifiez les règles de pare-feu appliquées au niveau du serveur et de la base de données. Assurez-vous également que les connexions entre les ordinateurs clients et le serveur et la base de données qui vient d’être restaurée sont activées. Pour en savoir plus, voir [Pare-feu de base de données SQL Azure](https://msdn.microsoft.com/library/azure/ee621782.aspx) et [Procédure : configurer les paramètres de pare-feu (Base de données SQL Azure)](https://msdn.microsoft.com/library/azure/jj553530.aspx).

## Vérifier les connexions de serveur et les utilisateurs des bases de données
Vérifiez que toutes les connexions utilisées par votre application existent sur le serveur qui héberge votre base de données restaurée. Créez à nouveau les connexions manquantes et accordez-leur les autorisations appropriées sur la base de données restaurée. Pour en savoir plus, voir [Gestion des bases de données et des connexions dans la base de données SQL Microsoft Azure](https://msdn.microsoft.com/library/azure/ee336235.aspx).

Vérifiez que les utilisateurs de chaque base de données, dans la base de données restaurée, sont bien associés à une connexion de serveur valide. Utilisez l’instruction ALTER USER pour mapper l’utilisateur à une connexion de serveur valide. Pour en savoir plus, voir [ALTER USER](http://go.microsoft.com/fwlink/?LinkId=397486).

## Configurer les alertes de télémétrie
Vérifiez que les paramètres de vos règles d’alertes existantes sont mappés à la base de données restaurée. Mettez à jour le paramètre dans l’un des cas suivants :

* La base de données restaurée utilise un nom différent de celui de la base de données source.
* La base de données restaurée se trouve sur un serveur différent du serveur source.

Pour en savoir plus, voir [Procédure : recevoir des notifications d’alerte et gérer des règles d’alerte dans Azure](https://msdn.microsoft.com/library/azure/dn306638.aspx).

## Activer la fonction d’audit
Si la fonction d’audit doit accéder à votre base de données, vous devez l’activer après la restauration de la base de données. Un bon indicateur de la nécessité d’activer l’audit est l’utilisation, par les applicatives clientes, de chaînes de connexion sécurisées dans un modèle *.database.secure.windows.net. Pour en savoir plus, voir [Prise en main de l’audit de base de données SQL](sql-database-auditing-get-started.md).

<!---HONumber=AcomDC_0803_2016-->