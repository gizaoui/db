# SQL

<br>

## 1. Tables temporaires

<br>

### 1.1.  Différence entre une table temporaire locale et une table temporaire globale

Dans SQL Server, les tables temporaires globales sont visibles par toutes les sessions (connexions).

<br>

### 1.2. Ex1

```sql
CREATE PROCEDURE #Niveau1
AS
  -- création de la table
  CREATE TABLE #test (ID INT IDENTITY, Code VARCHAR(50))
	
  -- remplissage de la table
  INSERT INTO #test (Code)
  SELECT 'PORTFOLIO'
 
  -- execution d'une sous procédure
  EXEC #Niveau2

  -- visualisation des données
  SELECT ID, Code  FROM #test
GO
```

```sql
CREATE PROCEDURE #Niveau2
AS
  -- création de la table
  CREATE TABLE #test (ID INT IDENTITY, Code VARCHAR(50))

  -- remplissage de la table
  INSERT INTO #test (Code)
        SELECT 'THIRDPARTY'
  UNION SELECT 'CLUSTER'
  UNION SELECT 'ENTITY'
GO
```

Les 2 procédures ayant été créées au préalable, j'exécute la procédure #niveau1. 
Qu'obtiendrai-je comme résultat et expliquer pourquoi ?

Donner l'ID, et le Code pour chaque ligne lorsqu'il y a un retour

Traitement équivalent.

```sql
-- création de la table
DROP TABLE IF EXISTS "test";
CREATE TABLE test (ID INTEGER PRIMARY KEY AUTOINCREMENT, Code TEXT);
	
-- remplissage de la table
INSERT INTO test (Code) SELECT 'PORTFOLIO';

 -- execution d'une sous procédure
 -- EXEC #Niveau2
 
DELETE FROM test;
 
INSERT INTO test (Code)
      SELECT 'THIRDPARTY'
UNION SELECT 'CLUSTER'
UNION SELECT 'ENTITY';

-- visualisation des données
SELECT ID, Code  FROM test;
```

<br>

### 1.3. Ex2


```sql
CREATE PROCEDURE #Niveau1
AS
  -- création de la table
  CREATE TABLE #test (ID INT IDENTITY, Code VARCHAR(50))
	
  -- remplissage de la table
  INSERT INTO #test (Code)
  SELECT 'PORTFOLIO'
 
  -- execution d'une sous procédure
  EXEC #Niveau2

  -- visualisation des données
  SELECT ID, Code FROM #test
GO
```

```sql
CREATE PROCEDURE #Niveau2
AS
  -- Vidange de la table
  TRUNCATE TABLE #test

  -- remplissage de la table
  INSERT INTO #test (Code)
        SELECT 'THIRDPARTY'
  UNION SELECT 'CLUSTER'
  UNION SELECT 'ENTITY'
GO
```

Les 2 procédures ayant été créées au préalable, j'exécute la procédure #niveau1.
Qu'obtiendrai-je comme résultat et expliquer pourquoi ?

Donner l'ID, et le Code pour chaque ligne lorsqu'il y a un retour

Traitement équivalent.

```sql
-- création de la table
DROP TABLE IF EXISTS "test";
CREATE TABLE test (ID INTEGER PRIMARY KEY AUTOINCREMENT, Code TEXT);
	
-- remplissage de la table
INSERT INTO test (Code) SELECT 'PORTFOLIO';

 -- execution d'une sous procédure
 -- EXEC #Niveau2
 
-- recréation de la table
DROP TABLE IF EXISTS "test";
CREATE TABLE test (ID INTEGER PRIMARY KEY AUTOINCREMENT, Code TEXT);
 
INSERT INTO test (Code)
      SELECT 'THIRDPARTY'
UNION SELECT 'CLUSTER'
UNION SELECT 'ENTITY';

-- visualisation des données
SELECT ID, Code  FROM test;
```

---

<br>

## 2. Case

```sql
DECLARE @test VARCHAR(50) = 'PORTFOLIO_TRANSACTION'
SELECT CASE WHEN @test LIKE 'TRANSACTION%' THEN 1
WHEN @test LIKE 'PORTFOLIO%' THEN 2
WHEN @test LIKE 'PORTFOLIO%TRANSACTION' THEN 3
ELSE 4 END
```

La requête analogue a été testée avec la base de données [SQLite](https://sqlitebrowser.org/) 
avec la release [3-13-1](https://sqlitebrowser.org/blog/version-3-13-1-released/).

Requête :

Le caractère **%** correspond à une chaîne de caratère quelconque.

```sql
SELECT CASE WHEN test LIKE 'TRANSACTION%' THEN 1
WHEN test LIKE 'PORTFOLIO%' THEN 2
WHEN test LIKE 'PORTFOLIO%TRANSACTION' THEN 3
ELSE 4 END "Resut"
FROM (SELECT 'PORTFOLIO_TRANSACTION' "test");
```

Resultat :

La requête renvoie la première correspondance trouvée.

|Resut|
| - |
| 2 | 

---

<br>

## 3. Clauses SQL

### 3.1. Ex1

Dans quel apparaissent les clauses ci-dessous dans un SELECT ?

| Clauses  | Ordre |
|   -      | -     |
| FROM     |  2    |
| ORDER BY |  6    |
| HAVING   |  5    |
| GROUP BY |  4    |
| SELECT   |  1    |
| WHERE    |  3    |

<br>

### 3.2. Ex2

Dans quel ordre sont évaluées, par le moteur SQL, les clauses ci-dessous, dans un SELECT ?

| Clauses  | Ordre |
|   -      | -     |
| FROM     |      |
| ORDER BY |      |
| HAVING   |      |
| GROUP BY |      |
| SELECT   |      |
| WHERE    |      |

<br>

### 3.3. Ex3

On a une table

```sql
#account (ID INT, AccountName VARCHAR(200))
```

Et un paramètre @userFilter de type INT pouvant contenir un ID de la table #account<br>
Écrire une requête qui nous retourne le contenu de la table #account filtrée sur la variable @userFilter 
que sa valeur soit définie ou non. Tous les enregistrements devront être retournés si la valeur est NULL.

---

<br>

## 4. Différence entre DELETE et un TRUNCATE

```sql
CREATE TABLE #test (ID INT IDENTITY, Code VARCHAR(50))

-- remplissage de la table
INSERT INTO #test (Code)
SELECT 'PORTFOLIO'

-- Début de la transaction
BEGIN TRAN Transaction1

-- vidange de la table
TRUNCATE TABLE #test

-- remplissage de la table
INSERT INTO #test (Code)
SELECT 'CLUSTER'

ROLLBACK

-- visualisation des données
SELECT ID, Code FROM #test
```

Que vais-je obtenir, à l'exécution du script ci-dessus et pourquoi ?

Donner l'ID, et le Code pour chaque ligne lorsqu'il y a un retour

---

<br>

## 5. Jointures

Les requêtes ont été testées avec la base de données [SQLite](https://sqlitebrowser.org/) 
avec la release [3-13-1](https://sqlitebrowser.org/blog/version-3-13-1-released/).

Les noms des tables et des champs ci-dessous ont été armonisés/re-écrite afin de faciliter la compréhension.

```sql
CREATE TABLE #Table1(ID INT, Code VARCHAR(50))
CREATE TABLE #Table2(ID INT, DefaultLabel VARCHAR(200))
CREATE TABLE #Table3(ID INT, DefaultDescription VARCHAR(1000))
```



### 5.1. Diagrammes à deux cercle

Tous les cas de figure seront traités.

**Pas uniquement ceux du test :**

- [Données communes entre les deux table](https://github.com/gizaoui/db/blob/main/SQL.md#5123-donn%C3%A9es-communes-des-table-t1-et-t2)
- [Données de la table *T1*](https://github.com/gizaoui/db/blob/main/SQL.md#5121-donn%C3%A9es-de-la-table-t1)
- [Données exclusivement liées à la table T1 ou à la table T2](https://github.com/gizaoui/db/blob/main/SQL.md#5127-donn%C3%A9es-exclusivement-li%C3%A9es-%C3%A0-la-table-t1-ou-%C3%A0-la-table-t2)


#### 5.1.1. Création des tables

```sql
DROP TABLE IF EXISTS "T1";
DROP TABLE IF EXISTS "T2";

CREATE TABLE "T1" ("ID"	INTEGER,"STR" TEXT);
CREATE TABLE "T2" ("ID"	INTEGER,"STR" TEXT);

INSERT INTO T1 VALUES (1, 'A'), (2, 'B'), (3, 'C'), (4, 'D'), (5, 'E'), (6, 'F'), (7, 'G');
INSERT INTO T2 VALUES (8, 'H'), (9, 'I'), (3, 'J'), (4, 'K'), (5, 'L'), (13, 'M'), (14, 'N');
```

##### 5.1.1.1. Association

Les données communes sont associées deux à deux pour éviter une duplication liée aux produits cartésiens.

| T1.ID | T1.STR | T2.ID | T2.STR |
| - | - | -  | - |
| 1 | A |  8 | H |
| 2 | B |  9 | I |
| 3 | C |  3 | J |
| 4 | D |  4 | K |
| 5 | E |  5 | L |
| 6 | F | 13 | M |
| 7 | G | 24 | N |

Données communes.

| ID | T1 | T2 |
| - | - | - |
| 3 | C | J |
| 4 | D | K |
| 5 | E | L |


Répartition des données sur un diagramme de *Venn*.

![00](pic/00.png)

<br>

---

<br>

#### 5.1.2. Sélection des données

Les trois espaces du diagramme induisent $\frac{1 - 2^{3+1}}{1 - 2}=7$ combinaisons possibles.

##### 5.1.2.1. Données de la table *T1*

![01](pic/01.png)

Requête :

```sql
SELECT T1.STR "T1", T2.STR "T2" 
FROM T1 
LEFT JOIN T2 ON T1.ID=T2.ID 
ORDER BY T1.STR, T2.STR;
```

Résultat :

| T1 | T2 |
| -  | -  |
| A  |    |
| B  |    |	
| C  | J  |
| D  | K  |
| E  | L  |
| F  |    |
| G  |    |

---

##### 5.1.2.2. Données de la table *T2*

![02](pic/02.png)

Requête :

```sql
SELECT T1.STR "T1", T2.STR "T2" 
FROM T1 RIGHT 
JOIN T2 ON T1.ID=T2.ID 
ORDER BY T1.STR, T2.STR;
```

Résultat :

| T1 | T2 |
| -  | -  |
|    |	H |
|    |	I |
| C  |	J |
| D  |	K |
| E  |	L |
|    |	M |
|    |	N |

---

##### 5.1.2.3. Données communes des table *T1* et *T2* 

![03](pic/03.png)

###### 5.1.2.3.1. Avec jointure 

Requête :

```sql
SELECT T1.STR "T1", T2.STR "T2" 
FROM T1 
INNER JOIN T2 ON T1.ID=T2.ID 
ORDER BY T1.STR, T2.STR;
```

Résultat :

| T1 | T2 |
| -  | -  |
| C  |	J |
| D  |	K |
| E  |	L |


###### 5.1.2.3.1. Sans jointure

Requête :

```sql
SELECT T1.ID, T1.STR
FROM T1
WHERE EXISTS (
  SELECT T2.ID
  FROM T2
  WHERE T1.ID=T2.ID
)
ORDER BY T1.STR;
```

Résultat :

| ID | STR |
| - | - |
| 3 | C |
| 4 | D |
| 5 | E |

---

##### 5.1.2.4. Données exclusivement liées à la table *T1*

![04](pic/04.png)

Requête :

```sql
SELECT T1.STR "T1", T2.STR "T2" 
FROM T1 LEFT 
JOIN T2 ON T1.ID=T2.ID 
WHERE T2.STR IS NULL 
ORDER BY T1.STR, T2.STR;
```

Résultat :

| T1 | T2 |
| -  | -  |
| A  |    |
| B  |    |
| F  |    |
| G  |    |

---

##### 5.1.2.5. Données exclusivement liées à la table *T2*

![05](pic/05.png)

Requête :

```sql
SELECT T1.STR "T1", T2.STR "T2" 
FROM T1
RIGHT JOIN T2 ON T1.ID=T2.ID 
WHERE T1.STR IS NULL 
ORDER BY T1.STR, T2.STR;
```

Résultat :

| T1 | T2 |
| -  | -  |
|    |  H |
|    |  I |
|    |  M |
|    |  N |

---

##### 5.1.2.6. Totalité de la base de données

![06](pic/06.png)

Requête :

```sql
SELECT T1.STR "T1", T2.STR "T2" 
FROM T1 
FULL OUTER JOIN T2 ON T1.ID=T2.ID 
ORDER BY T1.STR, T2.STR;
```

Résultat :

| T1 | T2 |
| -  | -  |
|    |  H |
|    |  I |
|    |  M |
|    |  N |
| A  |    |
| B  |    |	
| C  |	J |
| D  |	K |
| E  |  L |
| F  |    |
| G  |    |	

---

##### 5.1.2.7. Données exclusivement liées à la table *T1* ou à la table *T2*

![06](pic/07.png)

Requête :

```sql
SELECT T1.STR "T1", T2.STR "T2" 
FROM T1 
FULL OUTER JOIN T2 ON T1.ID=T2.ID 
WHERE (T1.STR IS NULL OR T2.STR IS NULL) 
ORDER BY T1.STR, T2.STR;
```

Résultat :

| T1 | T2 |
| -  | -  |
|    |  H |
|    |  I |
|    |  M |
|    |  N |
| A  |    |
| B  |    |
| F  |    |	
| G  |    |	

---

<br>

### 5.2. Diagrammes à trois cercle


#### 5.2.1. Création des tables


```sql
DROP TABLE IF EXISTS "T1";
DROP TABLE IF EXISTS "T2";
DROP TABLE IF EXISTS "T3";

CREATE TABLE "T1" ("ID"	INTEGER,"STR" TEXT);
CREATE TABLE "T2" ("ID"	INTEGER,"STR" TEXT);
CREATE TABLE "T3" ("ID"	INTEGER,"STR" TEXT);

INSERT INTO T1 VALUES (1, 'A'), (2, 'B'), (3, 'C'), (4, 'D'),  (5, 'E'),  (6, 'F'), (7, 'G'), (8, 'H');
INSERT INTO T2 VALUES (9, 'I'), (10, 'J'), (11, 'K'), (4, 'L'), (5, 'M'), (6, 'N'), (15, 'O'), (16, 'P');
INSERT INTO T3 VALUES (17, 'Q'), (18, 'R'), (19, 'S'),  (7, 'T'), (8, 'U'), (6, 'V'), (15, 'W'), (16, 'X');
```

##### 5.2.1.1. Association

Les données communes sont associées deux à deux pour éviter une duplication liée aux produits cartésiens.

| T1.ID | T1.STR | T2.ID | T2.STR | T3.ID | T3.STR |
| - | - | -   | -  | -  | -  |
| 1 | A |  9  | I  | 17 | Q  |
| 2 | B | 10  | J  | 18 | R  |
| 3 | C | 11  | K  | 19 | S  |
| 4 | D |  4  | L  |  7 | T  |
| 5 | E |  5  | M  |  8 | U  |
| 6 | F |  6  | N  |  6 | V  |
| 7 | G | 15  | O  | 15 | W  |
| 8 | H | 16  | P  | 16 | X  |


Données communes aux table *T1* et *T2*.

| ID | T1 | T2 |
| - | - | -  |
| 4 | D | L  |
| 5 | E | M  |
| 6 | F | N  |

Données communes aux table *T1* et *T3*.

| ID | T1 | T3 |
| - | - | -  |
| 6 | F | V  |
| 4 | G | T  |
| 5 | H | U  |

Données communes aux table *T2* et *T3*.

| ID | T2 | T3 |
| -  | - | -  |
|  6 | N | V  |
| 15 | O | W  |
| 16 | P | X  |


Répartition des données sur un diagramme de *Venn*.

![08](pic/08.png)

<br>

---

<br>

#### 5.2.2. Sélection des données

Les sept espaces du diagramme induisent $\frac{1 - 2^{7+1}}{1 - 2}=127$ combinaisons possibles.

On en sélectionnera deux.


##### 5.2.2.1. Données de la table *T1*

![09](pic/09.png)

Requête :

```sql
SELECT T1.STR "T1", T2.STR "T2", T3.STR "T2"
FROM T1 
LEFT JOIN T2 ON T1.ID=T2.ID
LEFT JOIN T3 ON T1.ID=T3.ID
ORDER BY T1.STR, T2.STR, T3.STR;
```

Résultat :

| T1 | T2 | T3 |
| -  | -  | -  |
| A  |    |    |
| B  |    |    |
| C  |    |    |
| D  | L  |    |
| E  | M  |    |
| F  | N  | V  |
| G  |	  | T  |
| H  |    | U  |

---

##### 5.2.2.2. Union de l'intersection entre les tables *T1*, *T2* et de l'intersection entre les tables *T1*, *T3* 

![10](pic/10.png)

Requête :

```sql
SELECT T1.STR "T1", T2.STR "T2", T3.STR "T3"
FROM T1 
LEFT JOIN T2 ON T1.ID=T2.ID 
LEFT JOIN T3 ON T1.ID=T3.ID
WHERE (T2 IS NOT NULL OR T3 IS NOT NULL)
ORDER BY T1.STR, T2.STR, T3.STR;
```

Résultat :

| T1 | T2 | T3 |
| -  | -  | -  |
| D  | L  |    |
| E  | M  |    |
| F  | N  | V  |
| G  |    | T  |
| H  |    | U  |



### 5.3. Classement

#### 5.3.1. Création des tables

```sql
DROP TABLE IF EXISTS "VENTES";
CREATE TABLE "VENTES" ("ID" INTEGER ,"PRENOM" TEXT ,"NOM" TEXT, "MOIS" INTEGER, "CA" INTEGER);
INSERT INTO "VENTES" ("ID", "PRENOM", "NOM", "MOIS", "CA") VALUES 
(1, 'Hélene', 'Bonnet', 5, 2300),
(2, 'Marie', 'Martin', 5, 2400),
(3, 'Hélene', 'Bonnet', 6, 2700),
(4, 'Marie', 'Martin', 6, 2700),
(5, 'Alexandre', 'Durand', 6, 2900),
(6, 'Marie', 'Martin', 7, 1200),
(7, 'Hélene', 'Bonnet', 7, 1200),
(8, 'Alexandre', 'Durand', 7, 1000);
```

#### 5.3.2 Classement de données

Classement selon le CA décroissant avec ajout d'un *ROWNUM*.

Requête :

```sql
SELECT ID, PRENOM, NOM, MOIS, CA,
ROW_NUMBER () OVER ( ORDER BY CA DESC ) CLASSEMENT
FROM VENTES;
```


Résultat :

| ID | PRENOM | NOM | MOIS | CA | CLASSEMENT |
| - | -         | -      | - | -    | - |
| 5 | Alexandre | Durand | 6 | 2900 | 1 |
| 3 | Hélene    | Bonnet | 6 | 2700 | 2 |
| 4 | Marie     | Martin | 6 | 2700 | 3 |
| 2 | Marie     | Martin | 5 | 2400 | 4 |
| 1 | Hélene    | Bonnet | 5 | 2300 | 5 |
| 6 | Marie     | Martin | 7 | 1200 | 6 |
| 7 | Hélene    | Bonnet | 7 | 1200 | 7 |
| 8 | Alexandre | Durand | 7 | 1000 | 8 |








