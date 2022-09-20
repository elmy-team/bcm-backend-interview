# Objectif

L’objectif de cet exercice est de créer une application standalone qui récupère et agrège des données de productions de centrales électriques. On cherche à obtenir la somme des puissances de chaque centrale au pas de temps 15 min.

**Nous regarderons le code de la même façon que nous regarderions un code de production.**

# Contexte

Pour cet exercice, nous nous plaçons dans le contexte suivant: nous avons trois centrales de production d'énergie qui sont :

- Hawes
- Barnsley 
- Hounslow 

Sur chacune de ces centrales, nous avons des infrastructures de monitoring qui sont en capacité de remonter des informations basiques de production d’électricité : ici, nous nous intéressons uniquement à la **puissance** de production sur un pas de temps donné. Le pas de temps de mesure est toujours constant, mais diffère pour chaque centrale:

| **Centrale** | **Pas de temps** |
| ------------ | ---------------- |
| Hawes        | 15 minutes       |
| Barnsley     | 30 minutes       |
| Hounslow     | 60 minutes       |

Nous sommes seulement intéressés par la somme de ces puissances de production. Pour faciliter l’exercice, la somme de ces mesures par pas de temps se cale sur le plus petit de nos centrales, et toutes les centrales sont dans la même timezone. C’est bien ici la **somme** des puissances des centrales qui nous intéresse comme fonction d’agrégation.

Le chapitre suivant donne plus d’indications sur chacune des centrales.


# Centrales

Comme vous pourrez le voir dans la description de chaque centrale, les formats peuvent être différents. Chaque réponse contient pour chaque intervalle de temps, la valeur de la puissance mesurée sur le site de production.

Chaque centrale dispose d’une API permettant la récupération des données enregistrées sur le site. Chaque API prend deux paramètres: `from` et `to`.  `from` correspond au point de départ de la remontée d’informations, `to` étant le point final.

Ces APIs renvoient l’ensemble des données existantes comprises entre `from` et `to` sous forme de segment d’une durée fixe (15 min, 30 min, 60 min, cf début du document). Toutes les données temporelles sont renvoyées sous forme de timestamp.


## Hawes 

URL: https://interview.beta.bcmenergy.fr/hawes
Query Params:

| Parameter | Description                   | Format     |
| --------- | ----------------------------- | ---------- |
| from      | The start date for the search | DD-MM-YYYY |
| to        | The end date for the search   | DD-MM-YYYY |

**Exemple de requête et réponse**


    curl https://interview.beta.bcmenergy.fr/hawes?from=01-01-2020&to=05-01-2020
    
    [
       {
          "start":1578006000,
          "end":1578006900,
          "power":710
       },
       {
          "start":1578006900,
          "end":1578007800,
          "power":627
       },
       {
          "start":1578007800,
          "end":1578008700,
          "power":518
       },
       {
          "start":1578008700,
          "end":1578009600,
          "power":750
       },
       ...
    ]


## Barnsley

URL: https://interview.beta.bcmenergy.fr/barnsley
Query Params:

| Parameter | Description                   | Format     |
| --------- | ----------------------------- | ---------- |
| from      | The start date for the search | DD-MM-YYYY |
| to        | The end date for the search   | DD-MM-YYYY |

**Exemple de requête et réponse**


    curl https://interview.beta.bcmenergy.fr/barnsley?from=01-01-2020&to=05-01-2020
    
    [
       {
          "start_time":1578006000,
          "end_time":1578007800,
          "value":774
       },
       {
          "start_time":1578007800,
          "end_time":1578009600,
          "value":682
       },
       {
          "start_time":1578009600,
          "end_time":1578011400,
          "value":622
       },
       ...
    ]


## Hounslow

URL: https://interview.beta.bcmenergy.fr/hounslow
Query Params:

| Parameter | Description                   | Format     |
| --------- | ----------------------------- | ---------- |
| from      | The start date for the search | DD-MM-YYYY |
| to        | The end date for the search   | DD-MM-YYYY |

**Exemple de requête et réponse**


    curl https://interview.beta.bcmenergy.fr/hounslow?from=01-01-2020&to=05-01-2020
    
    debut,fin,valeur
    1578006000,1578009600,568
    1578009600,1578013200,770
    1578013200,1578016800,754
    


# Programme attendu

Comme décrit plus haut, nous attendons un programme qui soit en mesure de pouvoir agréger (en faisant la somme) les différentes valeurs remontées par chaque centrale.

Nous ne voulons pas d’API, mais un programme standalone écrit dans le langage de votre choix. Le nombre de frameworks utilisés doit être tenu au strict minimum.

Notre programme prend 3 paramètres en entrée:

- from (date au format DD-MM-YYYY)
- to (date au format DD-MM-YYYY)
- format (string: json | csv)

Où `from` correspond au point de départ de la remontée d’infos et `to` le point final. Une fois les agrégations finalisées, le programme doit afficher sur la sortie standard, le résultat en utilisant le format correspondant au paramètre passé en entrée de programme (paramètre `format`)

Le format peut être `json` ou `csv`. Même si dans le cadre de l’exercice, uniquement ces deux formats sont supportés, il faut prévoir que d’autres formats seront rajoutés.

De même, on sait dès à présent que le nombre de centrales prises en charge par le programme va fortement évoluer. Il faut donc l’avoir en tête dans le programme.

Quelque soit le format, on s’attend à avoir pour chaque intervalle de temps de 15 minutes la somme des valeurs des centrales.

Même si le format est libre, voici une possibilité de réponse au format json



    [
       {
          "start":"1578006000",
          "end":"1578006900",
          "power":2052,
       },
       {
          "start":"1578006900",
          "end":"1578007800",
          "power":1969
       },
    ...
    ]
    

## Notes
Attention: puissance =/= énergie.
Pour un changement de pas horaire, la puissance reste la même contrairement à une énergie qui serait divisée.
Ex: 10 Watt sur 20 minutes = 10 Watt sur 10 minutes et 10 Watt sur les 10 minutes suivantes.

## Contraintes

Il arrive fréquemment que les APIs de centrales aient des trous de données. Il n’existe jamais deux trous consécutifs. Dans le cas où un trou existe à la position `N` dans le retour d’une API, on utilise la moyenne des valeurs `N - 1` et `N + 1`. La première et la dernière valeur ne sont jamais manquantes. Cette interpolation doit se faire sur chaque retour unitaire de l’API est non sur l’agrégation. Un trou de données est facilement identifiable car l'objet correspondant au pas de temps concerné n'est pas présent dans la réponse.

Il est à noter que dans le cadre de notre test, les APIs des centrales renvoient des données **totalement aléatoires**.

De plus, il n’est pas rare que les APIs renvoient des erreurs 500. 


# Points d’attention

Comme mentionné en début d’énoncé, nous regarderons le code comme si il avait été écrit pour la production.
Nous devons être en mesure de lancer facilement le programme.


# Bonus

Ecrire un dockerfile pour pouvoir lancer facilement l’application.
