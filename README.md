# databazy_movielens

1. Úvod a popis projektu
Tento projekt sa zameriava na spracovanie a analýzu dát z MovieLens datasetu. Dáta pokrývajú informácie o používateľoch, filmoch, žánroch, hodnoteniach a tagoch, ktoré používatelia pridali k filmom. Cieľom je pochopiť správanie používateľov, trendy hodnotení filmov a preferencie žánrov.

2. Dátová architektúra
Východiskové dáta boli uložené v relačnom modeli, znázornenom na nasledovnom ERD diagrame:

Tieto dáta boli transformované do hviezdicového modelu pre efektívnu analytiku. Hviezdicový model obsahuje fact_ratings ako centrálnu tabuľku a dimenzie ako podporné tabuľky:

3. Vizualizácie
Graf 1: Počet hodnotení podľa dekády vydania filmu
Tento graf znázorňuje, ako sa menil počet hodnotení filmov podľa dekád vydania.

SQL Dotaz:

```sql
Copy code
SELECT
    dm.release_decade,
    COUNT(DISTINCT fr.movie_key) AS movie_count
FROM fact_ratings fr
JOIN dim_movies dm ON fr.movie_key = dm.moviesID
WHERE dm.release_decade IS NOT NULL
GROUP BY dm.release_decade
ORDER BY dm.release_decade;
```
<img width="479" alt="graf 1" src="https://github.com/user-attachments/assets/774d3e1e-6fea-408d-bc7a-9100794dafb8" />


Vizualizácia:```

Bar Chart: osi X = release_decade, osi Y = movie_count.
Graf 2: Priemerný rating podľa žánru
Tento graf zobrazuje priemerné hodnotenie pre jednotlivé žánre.

SQL Dotaz:

```sql
Copy code
SELECT
    dg.name AS genre_name,
    ROUND(AVG(fr.rating_value), 2) AS avg_rating,
    COUNT(*) AS total_ratings
FROM fact_ratings fr
JOIN dim_genre dg ON fr.genre_key = dg.genreId
GROUP BY dg.name
ORDER BY avg_rating DESC;
```
Vizualizácia:
<img width="602" alt="graf 2" src="https://github.com/user-attachments/assets/66738885-4e81-4039-82d3-ba476eebba48" />

Bar Chart: osi X = genre_name, osi Y = avg_rating, zoradené podľa hodnotenia.
Graf 3: Aktivita používateľov podľa vekovej skupiny
Vizualizácia zobrazuje, koľko hodnotení spravili používatelia v rôznych vekových skupinách.

SQL Dotaz:

```sql
Copy code
SELECT
    du.age_group,
    COUNT(*) AS total_ratings
FROM fact_ratings fr
JOIN dim_users du ON fr.user_key = du.UsersID
GROUP BY du.age_group
ORDER BY total_ratings DESC;
```
Vizualizácia:
<img width="482" alt="graf 3" src="https://github.com/user-attachments/assets/b54e6e18-053b-4f38-a7a6-26307c5cb9cf" />

Pie Chart alebo Bar Chart: osi X = age_group, osi Y = total_ratings.
Graf 4: Najlepšie hodnotené filmy (Top 10)
Tento graf zobrazuje 10 filmov s najvyšším priemerným hodnotením.

SQL Dotaz:

```sql
Copy code
SELECT
    dm.title AS movie_title,
    ROUND(AVG(fr.rating_value), 2) AS avg_rating,
    COUNT(*) AS total_ratings
FROM fact_ratings fr
JOIN dim_movies dm ON fr.movie_key = dm.moviesID
GROUP BY dm.title
HAVING COUNT(*) > 50
ORDER BY avg_rating DESC
LIMIT 10;
```
Vizualizácia:
<img width="479" alt="graf 4" src="https://github.com/user-attachments/assets/3381f8e7-dd2b-4f5c-b6a8-24febbe7599d" />

Bar Chart: osi X = movie_title, osi Y = avg_rating.


```SELECT
    dm.release_decade,
    COUNT(DISTINCT dm.moviesID) AS movie_count
FROM fact_ratings fr
JOIN dim_movies dm ON fr.movie_key = dm.moviesID
WHERE dm.release_decade != 'UnknownDecade'
GROUP BY dm.release_decade
ORDER BY dm.release_decade;
```
<img width="483" alt="graf 5" src="https://github.com/user-attachments/assets/a2675950-62ba-48ce-b08c-9d0f6d1a4325" />



Popis obrázkov:
1. Obrázok: Relačný model (ERD diagram)

<img width="603" alt="schema" src="https://github.com/user-attachments/assets/284a4072-333f-44f6-a8b1-6a81dfcd1172" />

Tento diagram znázorňuje relačný model pôvodných dát zo systému MovieLens. Dátový model je navrhnutý podľa princípov normalizácie, aby sa minimalizovala redundancia dát.
Tabuľky a ich popis:
users:

Obsahuje informácie o používateľoch: vek, pohlavie, poštové smerovacie číslo a odkaz na zamestnanie.
Prepojenie na tabuľku occupations cez occupation_id.
occupations:

Obsahuje zoznam zamestnaní používateľov.
movies:

Obsahuje základné údaje o filmoch: názov a rok vydania.
ratings:

Obsahuje hodnotenia filmov, ktoré vytvorili používatelia, spolu s časovým údajom hodnotenia.
Prepojenie na tabuľku users (kto hodnotil) a movies (čo bolo hodnotené).
tags:

Obsahuje tagy, ktoré používatelia priradili k filmom, spolu s časovým údajom vytvorenia.
genres_movies:

Bridging tabuľka, ktorá prepája filmy s ich žánrami (M:N vzťah medzi movies a genres).
genres:

Obsahuje názvy žánrov.
age_group:

Obsahuje definície vekových skupín.
2. Obrázok: Hviezdicový model (Star Schema)

<img width="592" alt="schema_star" src="https://github.com/user-attachments/assets/dbef7489-a263-49cb-8032-fc43aaa5546b" />

Tento diagram znázorňuje hviezdicový model, ktorý bol vytvorený na základe pôvodného relačného modelu. Model je optimalizovaný pre analytiku a je vhodný na vykonávanie OLAP operácií (online analytického spracovania).
Tabuľky a ich popis:
fact_ratings (faktová tabuľka):

Obsahuje hodnotenia filmov spolu s prepojeniami na všetky dimenzie.
Obsahuje nasledujúce kľúče:
dim_users_UsersID: Cudzí kľúč na dimenziu používateľov.
dim_date_dateId: Cudzí kľúč na dimenziu dátumov.
dim_movies_moviesID: Cudzí kľúč na dimenziu filmov.
dim_genre_genreId: Cudzí kľúč na dimenziu žánrov.
rated_at: Časové údaje o hodnotení.
dim_users:

Obsahuje informácie o používateľoch vrátane vekových skupín, zamestnaní a poštových smerovacích čísel.
dim_movies:

Obsahuje údaje o filmoch vrátane názvu a roku vydania.
dim_genre:

Obsahuje zoznam žánrov.
dim_date:

Obsahuje časové údaje (deň, mesiac, rok, štvrťrok) pre analýzy podľa dátumu.
dim_tags:

Obsahuje tagy, ktoré používatelia priradili k filmom, spolu s časovým údajom ich vytvorenia.
Vzťah medzi diagramami:
ERD diagram predstavuje zdrojové dáta v normalizovanej podobe, ktoré sú vhodné na spracovanie, ale nie na rýchlu analytiku.
Hviezdicový model transformuje tieto dáta do denormalizovanej podoby, ktorá zjednodušuje analytické dotazy a vizualizácie.


1. Úvod a popis
Skript implementuje ETL proces pre MovieLens dataset v prostredí Snowflake. Tento proces zahŕňa vytvorenie staging tabuliek, načítanie dát z CSV súborov a transformáciu dát do hviezdicového modelu pre účely analytiky a vizualizácie. Dátová štruktúra je optimalizovaná na viacdimenzionálne analýzy.

2. Fáza: Extract (Načítanie dát do staging tabuliek)
Vytváranie staging tabuliek
Na začiatku sú vytvorené staging tabuľky pre jednotlivé entity zo zdrojového datasetu. Napríklad:

users_staging: Obsahuje údaje o používateľoch (vek, pohlavie, zamestnanie, PSČ).
movies_staging: Obsahuje údaje o filmoch (názov, rok vydania).
ratings_staging: Obsahuje hodnotenia používateľov vrátane časových údajov.
tag_staging: Obsahuje tagy priradené používateľmi k filmom.
Každá staging tabuľka bola vytvorená pomocou SQL príkazov, napr.:

```sql
Copy code
CREATE TABLE movies_staging (
    id INT PRIMARY KEY,
    title VARCHAR(255),
    release_year CHAR(4)
);
```
Načítanie dát do staging tabuliek
CSV súbory sú nahraté do Snowflake cez internal stage pomocou príkazu COPY INTO:

```sql
Copy code
COPY INTO movies_staging
FROM @my_stage/movies.csv
FILE_FORMAT = (TYPE='CSV' FIELD_OPTIONALLY_ENCLOSED_BY='"' SKIP_HEADER=1);
```
3. Fáza: Transformácia dát (Transform)
Dimenzionálne tabuľky
Dáta zo staging tabuliek sú transformované a rozdelené do dimenzií:

dim_users:

Obsahuje údaje o používateľoch vrátane vekových kategórií (napr. „18-24“) a zamestnania.
Príklad transformácie veku do kategórií:
```sql
Copy code
CASE 
    WHEN u.age < 18 THEN 'Under 18'
    WHEN u.age BETWEEN 18 AND 24 THEN '18-24'
    ...
END AS age_group
dim_movies:
```
Obsahuje údaje o filmoch vrátane dekád vydania (napr. „1990s“):
```sql
Copy code
CASE
    WHEN TRY_TO_NUMBER(m.release_year) BETWEEN 1990 AND 1999 THEN '1990s'
    ...
END AS release_decade
dim_genre:
```
Obsahuje zoznam žánrov filmov.
dim_date:

Obsahuje dátumy hodnotení, štruktúrované podľa dní, mesiacov a rokov:
```sql
Copy code
SELECT
    ROW_NUMBER() OVER (ORDER BY CAST(r.rated_at AS DATE)) AS dateId,
    CAST(r.rated_at AS DATE) AS date_value,
    ...
dim_tags:
```
Obsahuje unikátne tagy pridelené k filmom spolu s časovými údajmi o ich vytvorení.
Faktová tabuľka: fact_ratings
Faktová tabuľka centralizuje hodnotenia a prepojenia na všetky dimenzie:

Kľúčové atribúty:
rating_key: Primárny kľúč hodnotenia.
user_key, movie_key, genre_key, date_key: Odkazy na dimenzie.
rating_value: Skutočné hodnotenie (1–5 hviezdičiek).
rated_at, rated_timestamp: Časové údaje o hodnotení.
Príklad pripojenia dimenzií:

```sql
Copy code
SELECT
    r.id AS rating_key,
    r.rating AS rating_value,
    du.UsersID AS user_key,
    dm.moviesID AS movie_key,
    dg.genreId AS genre_key,
    dd.dateId AS date_key,
    CAST(r.rated_at AS DATE) AS rated_at
FROM ratings_staging r
JOIN dim_users du ON r.user_id = du.UsersID
JOIN dim_movies dm ON r.movie_id = dm.moviesID
LEFT JOIN genres_movies_staging gms ON r.movie_id = gms.movie_id
LEFT JOIN dim_genre dg ON gms.genre_id = dg.genreId
JOIN dim_date dd ON dd.date_value = CAST(r.rated_at AS DATE);
```
---
autor: Ondrej Smolárik
