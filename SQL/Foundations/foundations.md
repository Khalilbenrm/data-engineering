

# Rappel sur le langage SQL

## I - Définition des données

### 1 - Création des tables

```sql
CREATE TABLE nom_table (
    attribut1 TYPE,
    attribut2 TYPE,
    ...,
    contrainte_integrite_1,
    contrainte_integrite_2,
    ...
);
```

### 2 - Les types de données

```sql
-- Numériques
INTEGER            -- entier
DECIMAL(p, s)      -- nombre exact (p chiffres au total, s après la virgule)
FLOAT              -- nombre approché

-- Texte
CHAR(n)            -- chaîne de longueur fixe
VARCHAR(n)         -- chaîne de longueur variable (max n)
TEXT               -- texte long

-- Date et heure
DATE               -- AAAA-MM-JJ
TIME               -- hh:mm:ss
TIMESTAMP          -- date et heure

-- Autres
BOOLEAN            -- vrai / faux
```

### 3 - Les contraintes d'intégrité

Une contrainte d'intégrité est une règle que les données doivent toujours respecter. Le SGBD refuse toute insertion ou modification qui la viole.

Une contrainte peut être **nommée** avec `CONSTRAINT nom_contrainte`. C'est recommandé : le nom apparaît dans les messages d'erreur et permet de supprimer ou modifier la contrainte plus tard.

#### Clé primaire (`PRIMARY KEY`)

Identifie chaque ligne de façon unique. Elle est unique et ne peut pas être `NULL`. Une table n'a qu'une seule clé primaire.

```sql
CONSTRAINT nom_contrainte PRIMARY KEY (attribut)
```

Clé primaire composée de plusieurs attributs :

```sql
CONSTRAINT pk_ligne_commande PRIMARY KEY (id_commande, id_produit)
```

#### Clé étrangère (`FOREIGN KEY`)

Lie un attribut à la clé primaire (ou à un attribut unique) d'une autre table. Elle garantit qu'on ne référence pas une ligne qui n'existe pas.

```sql
CONSTRAINT nom_contrainte FOREIGN KEY (attribut)
    REFERENCES table_referencee (attribut_reference)
```

Comportement quand la ligne référencée est supprimée ou modifiée :

```sql
CONSTRAINT fk_commande_client FOREIGN KEY (id_client)
    REFERENCES client (id_client)
    ON DELETE CASCADE      -- supprime aussi les lignes liées
    ON UPDATE CASCADE      -- répercute la modification
```

| Option | Effet |
|---|---|
| `NO ACTION` / `RESTRICT` | Refuse l'opération (comportement par défaut) |
| `CASCADE` | Répercute la suppression ou la modification sur les lignes liées |
| `SET NULL` | Met l'attribut étranger à `NULL` |
| `SET DEFAULT` | Remet la valeur par défaut |

#### Unicité (`UNIQUE`)

Interdit deux valeurs identiques dans la colonne. Contrairement à la clé primaire, plusieurs contraintes `UNIQUE` sont possibles et, selon le SGBD, `NULL` peut être autorisé.

```sql
CONSTRAINT uq_client_email UNIQUE (email)
```

#### Valeur obligatoire (`NOT NULL`)

Interdit l'absence de valeur. S'écrit directement après le type de la colonne.

```sql
nom VARCHAR(50) NOT NULL
```

#### Condition (`CHECK`)

Impose qu'une condition soit vraie pour chaque ligne.

```sql
CONSTRAINT ck_commande_montant CHECK (montant >= 0)
CONSTRAINT ck_client_pays CHECK (pays IN ('France', 'Belgique', 'Suisse'))
```

#### Valeur par défaut (`DEFAULT`)

Valeur utilisée quand aucune n'est fournie à l'insertion. Ce n'est pas à proprement parler une contrainte d'intégrité, mais elle se déclare au même endroit.

```sql
pays VARCHAR(30) DEFAULT 'France'
```

#### Exemple complet

```sql
CREATE TABLE client (
    id_client  INTEGER,
    nom        VARCHAR(50)  NOT NULL,
    email      VARCHAR(100),
    pays       VARCHAR(30)  DEFAULT 'France',
    CONSTRAINT pk_client      PRIMARY KEY (id_client),
    CONSTRAINT uq_client_mail UNIQUE (email)
);

CREATE TABLE commande (
    id_commande    INTEGER,
    id_client      INTEGER       NOT NULL,
    date_commande  DATE          NOT NULL,
    montant        DECIMAL(10,2),
    CONSTRAINT pk_commande        PRIMARY KEY (id_commande),
    CONSTRAINT fk_commande_client FOREIGN KEY (id_client)
        REFERENCES client (id_client)
        ON DELETE CASCADE,
    CONSTRAINT ck_commande_montant CHECK (montant >= 0)
);
```

#### Ajouter ou supprimer une contrainte après coup

```sql
ALTER TABLE commande
    ADD CONSTRAINT ck_commande_date CHECK (date_commande >= '2020-01-01');

ALTER TABLE commande
    DROP CONSTRAINT ck_commande_date;
```

> L'ordre de création compte : on crée d'abord la table référencée (`client`), puis la table qui la référence (`commande`).


