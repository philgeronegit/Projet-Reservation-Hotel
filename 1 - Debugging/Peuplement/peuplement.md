# Peuplement

## Exécuter le script _reservation_hotel-peuplement.sql_

L'erreur suivante s'affiche :

```SQl
#1452 - Cannot add or update a child row: a foreign key constraint fails (`reservation_hotel`.`chambres`, CONSTRAINT `chambres_ibfk_2` FOREIGN KEY (`id_hotel`) REFERENCES `hotels` (`id`) ON DELETE CASCADE ON UPDATE CASCADE)
```

Il faut modifier l'ordre de peuplement des tables pour que les tables avec clés étrangères ne référencent pas des ids non existants.

Par example il faut peupler `salles_de_bain` avant de peupler `chambre_types`.

Il manque également la colonne _description_ dans la table `chambre_types`, on peut modifier cette table avec cette requête :

```SQL
ALTER TABLE `chambre_types`
ADD COLUMN `description` VARCHAR(255) DEFAULT NULL AFTER `nom`;
```

Le champ _datDebut_ de la table de la table `tarifs` ne respecte pas la convention de nommage des autres colonnes il sera renommé _date_debut_ dans le script de création de la base.

Ce script sera également modifié pour effacer les tables dans le bon ordre si jamais on veut la re-créer alors qu'elle existe déjà :

```SQL
DROP TABLE IF EXISTS `chambres`;
DROP TABLE IF EXISTS `tarifs`;
DROP TABLE IF EXISTS `chambre_type_couchage`;
DROP TABLE IF EXISTS `chambre_types`;
DROP TABLE IF EXISTS `couchages`;
DROP TABLE IF EXISTS `hotels`;
DROP TABLE IF EXISTS `salles_de_bain`;
DROP TABLE IF EXISTS `utilisateurs`;
```

## Comparaison avec les données originelles fournies par le client

Pour comparer les données de la base avec les données client il faut exécuter cette requête :

```SQL
SELECT
    c.numero AS chambre,
    h.libelle,
    h.etoile,
    ct.description,
    c.commentaire
FROM
    chambres c
        JOIN
    hotels h ON h.id = c.id_hotel
        JOIN
    chambre_types ct ON ct.id = c.id_type
```

## Expliquer sur quel point il faut être vigilant

Il faut être vigilant sur plusieurs points:

1. vérifier que les colonnes du script de peuplement existent dans la base de données
2. vérifier que le format des dates est bien _YYYY-MM-DD_
3. vérifier que le type de données qui vont être insérées correspondent au type de la base de données

## Cas particulier

La chambre 2 de l’hôtel “Ski Hôtel” est en travaux. Cette chambre ne sera pas disponible à la réservation.

Pour traiter le fait qu'elle ne soit pas disponible actuellement on peut ajouter un champ _statut_ dans la table `chambres` pour indiquer si une chambre est disponible, en travaux, ou réservée. On peut utiliser des valeurs comme 'disponible', 'en travaux', 'réservée'.

```SQL
ALTER TABLE chambres
ADD COLUMN statut ENUM('disponible', 'en travaux', 'réservée') DEFAULT 'disponible';
```

Avant de permettre une réservation, on vérifie le statut de la chambre. Si le statut est 'en travaux', on empêche la réservation.

```SQL
SELECT * FROM chambres
WHERE statut = 'disponible';
```
