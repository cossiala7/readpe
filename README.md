🚍 Projet ETL MacroBus - Data Warehouse

Ce projet implémente un flux ETL (Extract, Transform, Load) complet pour l'entreprise MacroBus. L'objectif est de migrer des données d'une base transactionnelle (Source) vers un entrepôt de données (DW) structuré en schéma en étoile.
📋 Prérequis

Avant de commencer, assurez-vous d'avoir installé les outils suivants :

    Docker Desktop (pour l'instance SQL Server)

    SQL Server Management Studio (SSMS)

    Visual Studio avec l'extension SSIS (SQL Server Integration Services)

    Power BI Desktop

🚀 Guide de déploiement (Étape par étape)
1. Lancement de l'infrastructure

Démarrez le conteneur SQL Server à l'aide de Docker Compose :
Bash

docker-compose up -d

    Note : Assurez-vous que le port 1433 est libre sur votre machine hôte.

2. Préparation des Bases de Données

Ouvrez SSMS et exécutez les scripts SQL dans cet ordre précis pour initialiser l'environnement :

    MacroBus_Source.sql : Création des tables transactionnelles.

    MacroBus_DW.sql : Création du schéma de l'entrepôt (Dimensions et Faits).

    Alimentation_Source_100.sql : Génération de 100 lignes de ventes pour les tests.

    Chargement_Dim_Temps.sql : Remplissage du calendrier (période 2003-2028).

        Important : Utilisez la version incluant DATEPART pour éviter les erreurs de conversion sur la colonne Jour.

3. Exécution du flux ETL (SSIS)

Ouvrez la solution SSIS dans Visual Studio :

    Gestionnaires de connexion : Vérifiez que les connexions OLE DB pointent correctement vers votre localhost.

    Ordre d'exécution des packages :

        Exécutez d'abord Chargement_Dim_Commercial et Chargement_Dim_Produit.

        Exécutez ensuite Chargement_Fait_Ventes.

    Validation : Le flux Fait_Ventes doit confirmer le traitement de 100 lignes avec succès.

4. Analyse dans Power BI

Connectez Power BI Desktop à votre base de données MacroBus_DW :

    Importation : Chargez les tables Fait_Ventes, Dim_Commercial, Dim_Produit et Dim_Temps.

    Modélisation : Vérifiez que les relations sont basées sur les Clés Techniques (Cle_Commercial, Cle_Produit, Code_Temps) et non sur les IDs sources.

🛠️ Dépannage (FAQ)
Problème	Solution
Erreur "Row yielded no match" (Lookup)	Vérifiez que Chargement_Dim_Temps.sql a été exécuté dans la base DW. Les dates récentes (2025/2026) doivent exister dans le référentiel.
Erreur de troncature (Msg 4712)	SQL bloque le TRUNCATE à cause des clés étrangères. Utilisez DELETE FROM Fait_Ventes puis DELETE FROM Dim_Temps pour vider les tables.
Doublons dans le cache SSIS	Videz les tables dimensions dans le DW avant de relancer le package pour éviter que le Lookup ne trouve plusieurs correspondances pour un même ID source.
