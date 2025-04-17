# Historisation

## Trigger qui enregistre une date de mise à jour pour un hotel

Il faut d'abords ajouter une colonne date_modification à la table `hotels`

```SQL
ALTER TABLE hotels
ADD COLUMN date_modification TIMESTAMP NULL DEFAULT NULL;
```

```SQL
DELIMITER //

CREATE TRIGGER trigger_hotel_date_maj
BEFORE UPDATE ON hotels
FOR EACH ROW
BEGIN
    SET NEW.date_modification = CURRENT_TIMESTAMP;
END//

DELIMITER ;
```

## Trigger qui enregistre l’ancienne version de l’enregistrement dans une table historique, à chaque modification

```SQL
CREATE TABLE histo_hotel (
    id INT UNSIGNED NOT NULL,
    libelle VARCHAR(50) NOT NULL,
    etoile VARCHAR(5) NOT NULL,
    date_modification DATETIME DEFAULT NULL,
    modification_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

```SQL
DELIMITER //

CREATE TRIGGER trigger_hotel_historique
BEFORE UPDATE ON hotels
FOR EACH ROW
BEGIN
    INSERT INTO histo_hotel (id, libelle, etoile, date_modification, modification_timestamp)
    VALUES (OLD.id, OLD.libelle, OLD.etoile, OLD.date_modification, CURRENT_TIMESTAMP);
END//

DELIMITER ;
```

## Peut-on ajouter deux triggers à une même table ?

Oui, il est tout à fait possible d'ajouter plusieurs triggers à une même table dans une base de données MySQL. On peut définir plusieurs triggers pour différents événements (INSERT, UPDATE, DELETE) et pour différents moments (BEFORE ou AFTER l'événement)

## Peut-on ajouter deux triggers à une même table et sur un même événement ?

Oui, il est possible d'ajouter plusieurs triggers à une même table pour le même événement (INSERT, UPDATE, ou DELETE) dans MySQL.

A noter que MySQL ne garantit pas l'ordre d'exécution des triggers pour le même événement et le même moment (BEFORE ou AFTER).
