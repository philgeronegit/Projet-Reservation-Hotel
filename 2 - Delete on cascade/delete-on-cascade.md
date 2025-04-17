# Delete on cascade

## Essayer de supprimer un enregistrement

Vérifier et essayer de supprimer un enregistrement dans la table `hotels`.

```SQL
DELETE FROM hotels WHERE hotels.id = 1
```

Grâce à RESTRICT sur ON DELETE la suppression génère l'erreur suivante :

```SQL
#1451 - Cannot delete or update a parent row: a foreign key constraint fails (`reservation_hotel`.`chambres`, CONSTRAINT `chambres_ibfk_2` FOREIGN KEY (`id_hotel`) REFERENCES `hotels` (`id`) ON DELETE RESTRICT ON UPDATE CASCADE)
```

## Modification des contraintes de clés étrangères

Modifier les contraintes de clés étrangères dans la vue relationnelle pour passer à ON DELETE ON CASCADE.

Cette fois-ci la requête suivante est possible :

```SQL
DELETE FROM hotels WHERE hotels.id = 1
```

L'hotel avec id 1 est supprimé ET les chambres qui référencent l'hotel 1 sont également supprimées.

## test de l'option UPDATE ON CASCADE

Avec cette option si on modifie l'id de l'hotel 2 en 20 les id de la table `chambres` qui référencent cet hotel vont aussi avoir leur id passer de 1 à 20.

```SQL
UPDATE hotels SET id = 20 WHERE id = 2;
```

id, id_hotel, id_type, numero, commentaire
'1', 20, '1', '1', 'NB : En travaux'
'3', 20, '3', '3', NULL
'4', 20, '1', '4', NULL
'6', 20, '6', '6', NULL

## test de l'option DELETE ON NULL

Avec cette option si on supprime un hotel les chambres qui référencent cet hotel auront la valeur de leur champ hotel_id à NULL.

id, id_hotel, id_type, numero, commentaire
'1', NULL, '1', '1', 'NB : En travaux'
'3', NULL, '3', '3', NULL
'4', NULL, '1', '4', NULL
'6', NULL, '6', '6', NULL

## Contrainte CASCADE et RESTRICT

Si la contrainte entre chambres et hôtels est en cascade et celle entre hôtels et tarifs est en RESTRICT.

### Analyse des contraintes

#### Cascade entre chambres et hôtels

Cela signifie que si un enregistrement dans la table hôtels est supprimé, les enregistrements correspondants dans la table chambres seront également supprimés automatiquement.

#### Restriction entre hôtels et tarifs

Cela signifie que si un enregistrement dans la table hôtels est référencé dans la table tarifs, la suppression de cet enregistrement dans hôtels ne sera pas permise.

Conclusion

La suppression d'un enregistrement dans la table hôtels n'est permise que si aucun enregistrement dans la table tarifs ne le référence. Si un enregistrement dans tarifs référence l'enregistrement à supprimer dans hôtels, la suppression sera bloquée en raison de la contrainte de restriction.

Donc, la suppression est permise uniquement si aucune référence n'existe dans la table tarifs pour l'enregistrement à supprimer dans hôtels.

```SQL
DELETE FROM hotels WHERE hotels.id = 1;

MySQL said:

#1451 - Cannot delete or update a parent row: a foreign key constraint fails (`reservation_hotel`.`tarifs`, CONSTRAINT `tarifs_ibfk_1` FOREIGN KEY (`id_hotel`) REFERENCES `hotels` (`id`) ON DELETE RESTRICT ON UPDATE CASCADE)
```
