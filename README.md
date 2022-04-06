Homework #3
Spring 2022,
Author: Joao Tomazoni
=====================

Exercise 1.
```SQL
CREATE TABLE Actor (
    A VARCHAR(20) PRIMARY KEY NOT NULL,
    ActorName VARCHAR(20),
    Country VARCHAR(20),
    DBIRTH DATE
);
```

Exercise 2.
```SQL
INSERT INTO Actor (A, ActorName, Country, DBIRTH)
VALUES ('A1', 'Daniel Radcliffe', 'Great Britain', '1989-07-02'),
       ('A2', 'Emma Watson', 'Great Britain', '1990-04-15');
```

Exercise 3.
```SQL
CREATE TABLE Movie (
    M VARCHAR(20) PRIMARY KEY NOT NULL,
    Title VARCHAR(20),
    Director VARCHAR(20),
    DirectorRegion VARCHAR(20) NOT NULL,
    MovieRegion VARCHAR(20) NOT NULL,
    CONSTRAINT tuple UNIQUE (Title, MovieRegion),
    CONSTRAINT check_region CHECK ((MovieRegion = 'Europe') OR (MovieRegion <> 'Europe' AND DirectorRegion = 'America'))
);
```

Exercise 4.
```SQL
INSERT INTO Movie (M, Title, Director, DirectorRegion, MovieRegion)
VALUES ('M1', 'HP 1', 'Columbus', 'Europe', 'America');
> Fails because the constraint requires MovieRegion != Europe to have DirectorRegion = America

INSERT INTO Movie (M, Title, Director, DirectorRegion, MovieRegion)
VALUES ('M1', 'HP 4', 'Newell', 'Asia', 'America');
> Fails because the constraint requires MovieRegion != Europe to have DirectorRegion = America

INSERT INTO Movie (M, Title, Director, DirectorRegion, MovieRegion)
VALUES ('M3', 'LOTR 3', 'Jackson', 'Asia', 'Europe');
> Query OK, 1 row affected

INSERT INTO Movie (M, Title, Director, DirectorRegion, MovieRegion)
VALUES ('M4', 'HP 1', 'Columbus', 'Asia', 'America');
> Fails because the constraint requires MovieRegion != Europe to have DirectorRegion = America

INSERT INTO Movie (M, Title, Director, DirectorRegion, MovieRegion)
VALUES ('M5', 'HP 6', 'Yates', 'Asia', 'America');
> Fails because the constraint requires MovieRegion != Europe to have DirectorRegion = America
```

Exercise 5.
```SQL
CREATE TABLE ActorMovie (
    A VARCHAR(20) NOT NULL,
    M VARCHAR(20) NOT NULL,
    Cachet DECIMAL(6,2),
    MainActor VARCHAR(20) NOT NULL,
    PRIMARY KEY (A, M),
    FOREIGN KEY (A) REFERENCES Actor(A) ON DELETE CASCADE,
    FOREIGN KEY (M) REFERENCES Movie(M) ON DELETE CASCADE,
    CONSTRAINT cachet_check CHECK (Cachet > 0)
)
```

Exercise 6.
```SQL
SELECT count(DISTINCT MainActor) as MainActors
FROM ActorMovie
```

Exercise 7.
```SQL
SELECT Title FROM Movie WHERE Director = 'Yates'
```

Exercise 8.
```SQL
CREATE VIEW GlobalView
AS
SELECT
ActorMovie.A as A,
ActorMovie.M as M,
ActorMovie.Cachet as Cachet,
ActorMovie.MainActor as MainActor,
Actor.ActorName as ActorName,
Actor.Country as Country,
Movie.Title as Title,
Movie.Director as Director,
Movie.MovieRegion as MovieRegion
FROM ActorMovie
JOIN Actor on ActorMovie.A = ACTOR.A
JOIN Movie on ActorMovie.M = Movie.M;

SELECT * FROM GlobalView;
```

Exercise 9. (Assuming we're allowed to query from the GlobalView)
```SQL
SELECT Country, COUNT(ActorName) as Number
FROM GlobalView
WHERE Title LIKE 'HP%'
GROUP BY 1;
```

Exercise 10.
```SQL
SELECT DISTINCT Title
FROM GlobalView
WHERE Country = 'Great Britain' AND ActorName = MainActor
```

Alternatively (If we weren't allowed to query from GlobalView)
```SQL
SELECT DISTINCT title FROM Movie
JOIN ActorMovie AM on Movie.M = AM.M
JOIN Actor A on AM.A = A.A
WHERE Country = 'Great Britain' AND ActorName = MainActor
```

Exercise 11.
```SQL
SELECT ActorName, Director, SUM(Cachet) as TotalCachet
FROM GlobalView
WHERE ActorName = 'Emma Watson'
GROUP BY 1, 2
```

Exercise 12.
```SQL
WITH D_R as (
    SELECT M, Title
    FROM GlobalView
    WHERE ActorName = 'Daniel Radcliffe'
),
     R_G as (
    SELECT M, Title
    FROM GlobalView
    WHERE ActorName = 'Rupert Grint'
)
SELECT DISTINCT D_R.Title FROM D_R
INNER JOIN R_G on D_R.M = R_G.M
```

Exercise 13.
```SQL
UPDATE Movie
SET Director = 'Columbus EU'
WHERE Director = 'Columbus' and DirectorRegion = 'Europe';

UPDATE Movie
SET Director = 'Columbus US'
WHERE Director = 'Columbus' AND DirectorRegion = 'America';
```

Exercise 14.
```SQL
GRANT SELECT ON Movie TO Ribo
```

Exercise 15.
> When there's parent-child relationship between two tables with the child pointing to parent via foreign key, `ON DELETE CASCADE` means that deletion of parent row causes the related child row to be deleted. `ON DELETE SET NULL` keeps the child row in the database even if parent row is deleted, but set their value to NULL
