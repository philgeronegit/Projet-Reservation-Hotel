# Importation

## Démarche

1. Ouvrir phpmyadmin et créer la base de données _reservation_hotel_
2. Exécuter le script _reservation_hotel_exercice.sql_ après avoir sélectionné la base _reservation_hotel_

   L'erreur suivante s'affiche :

   ```SQL
   CREATE TABLE IF NOT EXISTS `chambres` ...

   #1072 - Key column 'numChambre' doesn't exist in table
   ```

   Le script crée un clé unique avec les champs _id_hotel_ et _numChambre_ hors la table _chambres_ ne contient pas ce champ mais le champ _numero_ donc il faut corriger le script :

   ```SQL
   CREATE TABLE IF NOT EXISTS `chambres` (
     `id` int(11) UNSIGNED NOT NULL AUTO_INCREMENT,
     `id_hotel` int(11)  NOT NULL,
     `id_type` int(11)  NOT NULL,
     `numero` varchar(6) NOT NULL,
     `commentaire` text DEFAULT NULL,
     PRIMARY KEY (`id`),
     UNIQUE KEY `id_hotel` (`id_hotel`,`numero`),
     KEY `FK_Chambres_Hotels` (`id_hotel`),
     KEY `FK_Chambres_TypesChambre` (`id_type`)
   )
   ```

   Nouvelle erreur :

   ```SQL
   Static analysis:

   2 errors were found during analysis.

   Missing comma before start of a new alter operation. (near "KEY" at position 108)
   Missing comma before start of a new alter operation. (near "KEY" at position 231)

   SQL query: Copy

   -- -- Contraintes pour la table `chambres` -- ALTER TABLE `chambres` ADD CONSTRAINT `chambres_ibfk_1` KEY (`id_type`) REFERENCES `chambre_types` (`id`) ON DELETE CASCADE ON UPDATE CASCADE, ADD CONSTRAINT `chambres_ibfk_2` KEY (`id_hotel`) REFERENCES `hotels` (`id`) ON DELETE CASCADE ON UPDATE CASCADE;

   MySQL said: Documentation

   #1064 - You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'KEY (`id_type`) REFERENCES `chambre_types` (`id`) ON DELETE CASCADE ON UPDATE CA' at line 5
   ```

   Après avoir lu le manuel de référence MySQL pour ALTER TABLE [https://dev.mysql.com/doc/refman/8.4/en/alter-table.html] Il manque le mot clé FOREIGN avant KEY :

   ```SQL
    ALTER TABLE chambres
      ADD CONSTRAINT chambres_ibfk_1 FOREIGN KEY (id_type) REFERENCES chambre_types (id) ON DELETE CASCADE ON UPDATE CASCADE,
      ADD CONSTRAINT chambres_ibfk_2 FOREIGN KEY (id_hotel) REFERENCES hotels (id) ON DELETE CASCADE ON UPDATE CASCADE;
   ```

   et aussi :

   ```SQL
   ALTER TABLE `chambre_type_couchage`
     ADD CONSTRAINT `chambre_type_couchage_ibfk_1` FOREIGN KEY (`id_type`) REFERENCES `chambre_types` (`id`) ON DELETE CASCADE ON UPDATE CASCADE,
     ADD CONSTRAINT `chambre_type_couchage_ibfk_2` FOREIGN KEY (`id_couchage`) REFERENCES `couchages` (`id`) ON DELETE CASCADE ON UPDATE CASCADE;
   ```

3. Afficher le concepteur

   Après avoir affiché le diagramme les liaisons entre tables ne sont pas affichées.
   Il faut donc changer le moteur _MyISAM_ en _InnoDB_.
   Il y a aussi des incohérences entre l'id _id_hotel_ de la table _chambres_ qui est signé et l'id de la table _hotels_ qui est non signé. Il faut donc déclarer _id_hotel_ de la table _chambres_ en _UNSIGNED_ :

   ```SQL
   DROP TABLE IF EXISTS `chambres`;
   CREATE TABLE IF NOT EXISTS `chambres` (
     `id` int UNSIGNED NOT NULL AUTO_INCREMENT,
     `id_hotel` int UNSIGNED NOT NULL,
     `id_type` int UNSIGNED NOT NULL,
     `numero` varchar(6) NOT NULL,
     `commentaire` text DEFAULT NULL,
     PRIMARY KEY (`id`),
     UNIQUE KEY `id_hotel` (`id_hotel`,`numero`),
     KEY `FK_Chambres_Hotels` (`id_hotel`),
     KEY `FK_Chambres_TypesChambre` (`id_type`)
   ) ENGINE=InnoDB AUTO_INCREMENT=31 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
   ```

   Il y a encore d'autres incohérences de ce type (_UNSIGNED_ oubliés) dans le script qui ont été corrigées mais comme il s'agit de la même explication elles n'apparaissent pas ici (Voir le script _reservation_hotel_exercice.sql_ pour plus de détails).
