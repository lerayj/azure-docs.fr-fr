L’activité de copie dans Data Factory permet de copier les données d’un magasin de données source vers un magasin de données récepteur. Data Factory prend en charge les magasins de données suivants. Les données de n’importe quelle source peuvent être écrites dans n’importe quel récepteur. Cliquez sur une banque de données pour découvrir comment copier des données depuis/vers cette banque.

| Catégorie | Banque de données | Prise en charge en tant que source | Prise en charge en tant que récepteur |
|:--- |:--- |:--- |:--- |
| Microsoft Azure |[stockage d’objets blob Azure](../articles/data-factory/data-factory-azure-blob-connector.md) <br/> [Azure Data Lake Store](../articles/data-factory/data-factory-azure-datalake-connector.md) <br/> [Azure SQL Database](../articles/data-factory/data-factory-azure-sql-connector.md) <br/> [Azure SQL Data Warehouse](../articles/data-factory/data-factory-azure-sql-data-warehouse-connector.md) <br/> [Stockage Table Azure](../articles/data-factory/data-factory-azure-table-connector.md) <br/> [Azure DocumentDB](../articles/data-factory/data-factory-azure-documentdb-connector.md) <br/> [Index Recherche Azure](../articles/data-factory/data-factory-azure-search-connector.md)|✓ <br/> ✓  <br/> ✓  <br/> ✓  <br/> ✓  <br/> ✓  |✓  <br/> ✓  <br/> ✓  <br/> ✓  <br/> ✓  <br/> ✓  <br/> ✓ |
| Bases de données |[SQL Server](../articles/data-factory/data-factory-sqlserver-connector.md)\* <br/> [Oracle](../articles/data-factory/data-factory-onprem-oracle-connector.md)\* <br/> [MySQL](../articles/data-factory/data-factory-onprem-mysql-connector.md)\* <br/> [DB2](../articles/data-factory/data-factory-onprem-db2-connector.md)\* <br/> [Teradata](../articles/data-factory/data-factory-onprem-teradata-connector.md)\* <br/> [PostgreSQL](../articles/data-factory/data-factory-onprem-postgresql-connector.md)\* <br/> [Sybase](../articles/data-factory/data-factory-onprem-sybase-connector.md)\* <br/>[Cassandra](../articles/data-factory/data-factory-onprem-cassandra-connector.md)\* <br/>[MongoDB](../articles/data-factory/data-factory-on-premises-mongodb-connector.md)\*<br/>[Amazon Redshift](../articles/data-factory/data-factory-amazon-redshift-connector.md) |✓  <br/> ✓  <br/> ✓  <br/> ✓  <br/> ✓  <br/> ✓ <br/> ✓  <br/> ✓  <br/> ✓  <br/> ✓  |✓  <br/> ✓  <br/> &nbsp; <br/> &nbsp; <br/> &nbsp; <br/> &nbsp;<br/> &nbsp;<br/> &nbsp;<br/> &nbsp; <br/>&nbsp; |
| Fichier |[Système de fichiers](../articles/data-factory/data-factory-onprem-file-system-connector.md)\* <br/> [HDFS](../articles/data-factory/data-factory-hdfs-connector.md)\* <br/> [Amazon S3](../articles/data-factory/data-factory-amazon-simple-storage-service-connector.md) <br/> [FTP](../articles/data-factory/data-factory-ftp-connector.md) |✓ <br/> ✓  <br/> ✓  <br/> ✓  |✓  <br/> &nbsp;<br/>&nbsp; |
| Autres |[Salesforce](../articles/data-factory/data-factory-salesforce-connector.md)<br/> [ODBC générique](../articles/data-factory/data-factory-odbc-connector.md)\* <br/> [OData générique](../articles/data-factory/data-factory-odata-connector.md) <br/> [Table web (table HTML)](../articles/data-factory/data-factory-web-table-connector.md) <br/> [GE Historian](../articles/data-factory/data-factory-odbc-connector.md#ge-historian-store)* |✓ <br/> ✓  <br/> ✓  <br/> ✓  <br/> ✓ |&nbsp; <br/> &nbsp; <br/> &nbsp; <br/> &nbsp;<br/> &nbsp;<br/> &nbsp; |

> [!NOTE]
> Les banques de données signalées par un astérisque (*) peuvent être locales ou résider sur une instance Azure IaaS. Elles nécessitent que vous installiez une [passerelle de gestion des données](../articles/data-factory/data-factory-data-management-gateway.md) sur un ordinateur local ou Azure IaaS.
> 
> 



<!--HONumber=Nov16_HO2-->


