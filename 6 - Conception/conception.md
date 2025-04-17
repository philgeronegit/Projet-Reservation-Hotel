# Conception

Pour modéliser les réservations dans la base de données, il faut créer une nouvelle table qui enregistre les informations relatives aux réservations.

```SQL
CREATE TABLE reservations (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    id_chambre INT UNSIGNED NOT NULL,
    id_utilisateur INT NOT NULL,
    date_debut DATE NOT NULL,
    date_fin DATE NOT NULL,
    date_reservation TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    statut ENUM('confirmée', 'annulée', 'en attente') DEFAULT 'en attente',
    FOREIGN KEY (id_chambre) REFERENCES chambres(id),
    FOREIGN KEY (id_utilisateur) REFERENCES utilisateurs(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

## Trigger

Ce trigger vérifie si une nouvelle réservation chevauche une réservation existante pour la même chambre.
Il s'exécute avant l'insertion d'une nouvelle réservation et lève une erreur si un chevauchement est détecté.

```SQL
DELIMITER //

CREATE TRIGGER trigger_verif_chevauchement_reservation
BEFORE INSERT ON reservations
FOR EACH ROW
BEGIN
    -- Vérifie si la nouvelle réservation chevauche une réservation existante
    IF EXISTS (
        SELECT 1
        FROM reservations r
        WHERE r.id_chambre = NEW.id_chambre
          AND NOT (NEW.date_fin < r.date_debut OR NEW.date_debut > r.date_fin)
    ) THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Chevauchement de réservation détecté. Veuillez choisir une autre date.';
    END IF;
END//

DELIMITER ;
```

## Procédure stockée

Cette procédure stockée insère une nouvelle réservation dans la table reservations après avoir vérifié la disponibilité de la chambre.

La procédure prend en entrée l'ID de la chambre, l'ID du client, et les dates de début et de fin de la réservation.

```SQL
DELIMITER //

CREATE PROCEDURE creer_reservation(
    IN p_id_chambre INT,
    IN p_id_client INT,
    IN p_date_debut DATE,
    IN p_date_fin DATE
)
BEGIN
    -- Vérifie si la chambre est disponible pour les dates spécifiées
    IF NOT EXISTS (
        SELECT 1
        FROM reservations r
        WHERE r.id_chambre = p_id_chambre
          AND NOT (p_date_fin < r.date_debut OR p_date_debut > r.date_fin)
    ) THEN
        -- Insère la nouvelle réservation
        INSERT INTO reservations (id_chambre, id_client, date_debut, date_fin)
        VALUES (p_id_chambre, p_id_client, p_date_debut, p_date_fin);
    ELSE
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'La chambre n''est pas disponible pour les dates spécifiées.';
    END IF;
END//

DELIMITER ;
```

## Fonction

Cette fonction calcule le prix total d'une réservation en fonction des dates de début et de fin, et du tarif applicable.

La fonction prend en entrée l'ID de l'hôtel, l'ID du type de chambre, et les dates de début et de fin de la réservation.

```SQL
DELIMITER //

CREATE FUNCTION calculer_prix_total(
    p_id_hotel INT,
    p_id_type_chambre INT,
    p_date_debut DATE,
    p_date_fin DATE
) RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    DECLARE prix_total DECIMAL(10, 2);
    DECLARE tarif DECIMAL(7, 2);
    DECLARE nb_jours INT;

    -- Calcul du nombre de jours de la réservation
    SET nb_jours = DATEDIFF(p_date_fin, p_date_debut);

    -- Obtention du tarif applicable
    SELECT prix INTO tarif
    FROM tarifs
    WHERE id_hotel = p_id_hotel
      AND id_type = p_id_type_chambre
      AND dateDebut <= p_date_debut
    ORDER BY dateDebut DESC
    LIMIT 1;

    -- Calcul du prix total
    SET prix_total = tarif * nb_jours;

    RETURN prix_total;
END//

DELIMITER ;
```
