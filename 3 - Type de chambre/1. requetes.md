# Requêtes

## Requête 1

Préciser la salle de bain pour chaque type de chambre

```SQL
SELECT
    ct.id, ct.nom  as type, sdb.nom as sanitaire
FROM
    chambres c
        JOIN
    chambre_types ct ON c.id_type = ct.id
        JOIN
    salles_de_bain sdb ON ct.id_salle_de_bain = sdb.id
GROUP BY ct.nom
```

## Requête 2

Faire une requête qui permet d’affi cher aussi le couchage de chaque type de chambre

```SQL
SELECT
    ct.id,
    ct.nom AS type,
    sdb.nom AS sanitaire,
    ctc.qte,
    couchages.nom,
    couchages.nb_places
FROM
    chambres c
        JOIN
    chambre_types ct ON c.id_type = ct.id
        JOIN
    salles_de_bain sdb ON ct.id_salle_de_bain = sdb.id
        JOIN
    chambre_type_couchage ctc ON ct.id = ctc.id_type
        JOIN
    couchages ON ctc.id_couchage = couchages.id
GROUP BY ct.nom
```

## Requête 3

Afficher les résultats sur une seule ligne, en calculant le nombre de personnes

```SQL
SELECT
    ct.id,
    ct.nom AS type,
    sdb.nom AS sanitaire,
    GROUP_CONCAT(CONCAT(ctc.qte, 'x', c.nom)
        ORDER BY c.nom
        SEPARATOR ', ') AS details,
    SUM(ctc.qte * c.nb_places) AS nb_personnes
FROM
    chambre_types ct
        JOIN
    salles_de_bain sdb ON sdb.id = ct.id_salle_de_bain
        JOIN
    chambre_type_couchage ctc ON ct.id = ctc.id_type
        JOIN
    couchages c ON ctc.id_couchage = c.id
GROUP BY ct.id , ct.nom
ORDER BY ct.id;
```
