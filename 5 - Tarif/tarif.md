# Tarif

## Date de fin

Pour le tarif, on n'enregistre pas la date de fin en base de données, car elle est calculatoire. En effet, l’ancien tarif s'arrête là où le nouveau tarif commence.

```SQL
SELECT
    t1.id,
    t1.id_hotel,
    t1.id_type AS id_type_chambre,
    t1.date_debut,
    t1.prix,
    DATE_SUB((SELECT
                MIN(t2.date_debut)
            FROM
                tarifs t2
            WHERE
                t2.id_hotel = t1.id_hotel
                    AND t2.id_type = t1.id_type
                    AND t2.date_debut > t1.date_debut),
        INTERVAL 1 DAY) AS date_fin
FROM
    tarifs t1
WHERE
    id_hotel = 2 AND id_type = 2
ORDER BY t1.id_hotel , t1.id_type , t1.date_debut;
```

## Création d'une vue

```SQL
CREATE VIEW vue_tarif_date_fin AS
    SELECT
        t1.id,
        t1.id_hotel,
        t1.id_type AS id_type_chambre,
        t1.date_debut,
        t1.prix,
        DATE_SUB((SELECT
                    MIN(t2.date_debut)
                FROM
                    tarifs t2
                WHERE
                    t2.id_hotel = t1.id_hotel
                        AND t2.id_type = t1.id_type
                        AND t2.date_debut > t1.date_debut),
            INTERVAL 1 DAY) AS date_fin
    FROM
        tarifs t1
    WHERE
        id_hotel = 2 AND id_type = 2
    ORDER BY t1.id_hotel , t1.id_type , t1.date_debut;
```

## Créer une fonction get_tarif_jour

Cette fonction calcule le prix d’une chambre pour un jour donné (DATE), dans un hôtel donné et pour un type de chambre donné, en allant chercher le tarif valide dans la table tarifs.

```SQL
DELIMITER //

CREATE FUNCTION get_tarif_jour(hotel_id INT, type_chambre_id INT, jour DATE)
RETURNS DECIMAL(7,2)
DETERMINISTIC
BEGIN
    DECLARE tarif DECIMAL(7,2);

    SELECT prix INTO tarif
    FROM tarifs
    WHERE id_hotel = hotel_id
      AND id_type = type_chambre_id
      AND date_debut <= jour
    ORDER BY date_debut DESC
    LIMIT 1;

    RETURN tarif;
END//

DELIMITER ;
```

Example d'utilisation

```SQL
SELECT get_tarif_jour(2, 1, '2023-10-01') AS tarif_du_jour;
```

## Validation

Imaginons une règle métier sur le prix : les prix ne peuvent qu’augmenter, jamais diminuer.
Créer un trigger BEFORE UPDATE sur la table tarif qui provoque une erreur si le prix est inférieur ou égale aux autres tarifs et oblige une augmentation constante.

```SQL
DELIMITER //

CREATE TRIGGER trigger_check_prix_augmente
BEFORE UPDATE ON tarifs
FOR EACH ROW
BEGIN
    -- Check if the new price is less than or equal to the current price
    IF NEW.prix <= OLD.prix THEN
        -- Raise an error to prevent the update
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Le nouveau prix doit être supérieur au prix actuel.';
    END IF;
END//

DELIMITER ;
```
