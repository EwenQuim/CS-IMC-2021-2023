# TP Infrastructure

## SQL

**Exercice 0**: Décrivez les tables et les attributs.

```sql
exec sp_help
exec sp_columns tNames
exec sp_columns tPrincipals
exec sp_columns tTitles
```



**Exercice 1**: Visualisez l'année de naissance de l'artiste `Jude Law`.

```sql
select birthYear from [dbo].[tNames] where dbo.tnames.primaryName = 'Jude Law';
```

1972



**Exercice 2**: Comptez le nombre d'artistes présents dans la base de donnée.

```sql
select count(nconst) from [dbo].[tNames] ;
```

438361



**Exercice 3**: Trouvez les noms des artistes nés en `1960`, affichez ensuite leur nombre.

```sql
select primaryName from [dbo].[tNames] where dbo.tnames.birthYear = 1960;
select count(nconst) from [dbo].[tNames] where dbo.tnames.birthYear = 1960;
```

890

| primaryName |
| --------------------- |
| Antonio Banderas      |
| Kenneth Branagh       |
| Colin Firth           |
| Julianne Moore        |
| Kristin Scott Thomas  |
| Jean-Claude Van Damme |



**Exercice 4**: Trouvez l'année de naissance la plus représentée parmi les acteurs (sauf 0!), et combien d'acteurs sont nés cette année là.

```sql
select top 3 count(nconst), birthYear from [dbo].[tNames] where birthYear <> 0 group by birthYear order by count(nconst) DESC; 
```

| Nombre | Année |
| ------ | ----- |
| 1775   | 1980  |
| 1727   | 1979  |
| 1716   | 1978  |

Acteurs seulements

```sql
select count([dbo].[tNames].nconst), birthYear from [dbo].[tNames] 
join [dbo].[tPrincipals] on [dbo].[tNames].nconst = [dbo].[tPrincipals].nconst
where birthYear <> 0
and category like 'acted in'
group by birthYear order by count([dbo].[tNames].nconst) DESC; 
```

| Nombre | Année |
| ------ | ----- |
| 3486   | 1980  |
| 3420   | 1986  |
| 3366   | 1982  |



**Exercice 5**: Trouvez les artistes ayant joué dans plus d'un film

```sql
select dbo.tNames.primaryName, number
from dbo.tNames
inner join
(
    select nconst, count(nconst) as number
    from dbo.tPrincipals
    where category = 'acted in'
    GROUP BY nconst
    HAVING count(nconst) > 1
    
) AS temp
on dbo.tNames.nconst = temp.nconst
```

| primaryName | number |
| ----------- | ------ |
| Gong Li          | 5    |
| John Cleese      | 8    |
| Brad Pitt        | 12   |
| Woody Allen      | 2    |
| Gillian Anderson | 9    |
| Pamela Anderson  | 3    |



**Exercice 6**: Trouvez les artistes ayant eu plusieurs responsabilités au cours de leur carrière (acteur, directeur, producteur...).

```sql
select primaryName
from (
    select  nconst
    from tPrincipals
    group by nconst
    having count(distinct(category))>1
) as solution
join dbo.tNames
on dbo.tNames.nconst = solution.nconst;
```


| primaryName |
| --------------- |
| Brad Pitt       |
| Woody Allen     |
| Luc Besson      |
| Kenneth Branagh |
| Pierce Brosnan  |
| George Clooney  |



**Exercice 7**: Trouver le nom du ou des film(s) ayant le plus d'acteurs (i.e. uniquement *acted in*).

```sql
select tTitles.primaryTitle, actorsNumber
from tTitles
join (
    select tconst, count(nconst) as actorsNumber
    from tPrincipals
    where category = 'acted in'
    group by tconst
) as res
on res.tconst = tTitles.tconst
order by actorsNumber desc
```

| primaryTitle | actorsNumber |
| ----------------------------- | ---- |
| P.O.W. Open Air am Meer       | 10   |
| MacGuffin                     | 10   |
| Aakhri Pal                    | 10   |
| Hollywood Casting Confessions | 10   |
| Corazones en Llamas 3         | 10   |
| Corazones en Llamas 2         | 10   |



**Exercice 8**: Montrez les artistes ayant eu plusieurs responsabilités dans un même film (ex: à la fois acteur et  directeur, ou toute autre combinaison) et les titres de ces films.

```sql
select primaryName, primaryTitle
from (
    select  nconst, tconst
    from tPrincipals
    group by nconst, tconst
    having count(distinct(category))>1
) as solution
join dbo.tTitles
on tTitles.tconst = solution.tconst
join dbo.tNames
on dbo.tNames.nconst = solution.nconst;
```

| primaryName | primaryTitle |
| ----------- | ------------ |
| Samuthirakani    | Vinodhaya Sitham               |
| Brian George     | Refinery Surveyor Black        |
| Lom Harsh        | Yeh Hai India                  |
| Trista Suke      | Foxy                           |
| Chiwetel Ejiofor | The Boy Who Harnessed the Wind |
| Andy Barker      | Rest Area                      |



## Cypher

**Exercice 1**: Ajoutez une personne ayant votre prénom et votre nom dans le graphe. Verifiez qui le noeud a bien éte crée.

```cypher
CREATE(Clarence: Name {nconst: 'cccccccc' , primaryName:'Clarence Charles' , birthYear: 1999})
```

````cypher
MATCH (n:Name) WHERE n.primaryName = 'Clarence Charles' RETURN n
````



**Exercice 2**: Ajoutez un film nommé `L'histoire de mon 20 au cours Infrastructure de donnees`

```cypher
CREATE(myFilm: Title {tconst: 'llllllll' , primaryTitle:"L'histoire de mon 20 en Infra" , startYear: 2022})
```



**Exercice 3**: Ajoutez la relation `ACTED_IN` qui modélise votre participation à ce film en tant qu'acteur/actrice

```cypher
MATCH
  (actor:Name{primaryName: 'Clarence Charles'}),
  (film:Title{primaryTitle: "L'histoire de mon 20 en Infra"})
CREATE (actor)-[r:acted_in]->(film)
RETURN type(r)
```

````cypher
MATCH (n:Name{primaryName: 'Clarence Charles'})-[r]->(t:Title) RETURN n, r, t
````



**Exercice 4**: Ajoutez deux de vos professeurs/enseignants comme réalisateurs/réalisatrices de ce film.

````cypher
CREATE(Name {nconst: 'cabaretl' , primaryName:'Laurent Cabaret' , birthYear: 1980})  
````

````cypher
CREATE(Name {nconst: 'quercini' , primaryName:'Gianluca Quercini' , birthYear: 1980})
````

```cypher
MATCH
  (actor:Name),
  (film:Title{primaryTitle: "L'histoire de mon 20 en Infra"})
WHERE actor.nconst = 'cabaretl' OR actor.nconst = 'quercini'
CREATE (actor)-[r:directed]->(film)
RETURN type(r)    
```

```cypher
MATCH (n:Name)-[r]->(t:Title{tconst: 'llllllll'}) RETURN n, r, t
```



**Exercice 5**: Affichez le noeud représentant l'acteur nommé `Jude Law`, et visualisez son année de naissance.

```cypher
MATCH(n:Name{primaryName:'Jude Law'}) RETURN n.birthYear
```

1972



**Exercice 6**: Visualisez l'ensemble des films.

````cypher
MATCH(t:Title) return t limit 5000
````

Sans `limit`, la requête plante (timeout).



**Exercice 7**: Trouvez les noms des artistes nés en `1960`, affichez ensuite leur nombre.

````cypher
MATCH (n:Name{birthYear: 1960}) RETURN count(n)
````

890



**Exercice 8**: Trouver l'ensemble des acteurs (sans entrées doublons) qui ont joué dans plus d'un film.

````cypher
MATCH (u:Title)<-[:acted_in]-(n:Name)-[:acted_in]->(t:Title)
RETURN distinct n.primaryName limit 500
````



**Exercice 9**: Trouvez les artistes ayant eu plusieurs responsabilités au cours de leur carrière (acteur, directeur, producteur...).

````cypher
MATCH (n:Name)-[r]->(), (n:Name)-[t]->()
RETURN n
````



**Exercice 10**: Montrez les artistes ayant  eu plusieurs responsabilités dans un même film (ex: à la fois acteur et  directeur, ou toute autre combinaison) et les titres de ces films.

````cypher
MATCH (t:Title)<-[r]-(n:Name)-[s]->(t:Title)
WHERE type(r) <> type(s)
WITH DISTINCT n, t
RETURN n.primaryName, t.primaryTitle
````



**Exercice 11**: Trouver le nom du ou des film(s) ayant le plus d'acteurs.

````cypher
MATCH (actor:Name)-[:acted_in]->(movie:Title) 
RETURN 
    movie.primaryTitle,
    COLLECT(DISTINCT actor.nconst), 
    COUNT(DISTINCT actor) as COUNTING 
ORDER BY COUNTING DESC LIMIT 5;
````

Alone 65 actors

## TIL

- strings entre single quotes
- simple égal
- multiple select statements in SQL
- aggregators columns need to be renamed to be used as result for another table
