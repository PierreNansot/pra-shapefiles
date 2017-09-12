# Fichiers vecteurs Région agricoles

Selon le ministère de l'agriculture :

> Les régions agricoles et petites régions agricoles ont été définies (en 1946) pour mettre en évidence des zones agricoles homogènes. La Région Agricole (RA) couvre un nombre entier de communes formant une zone d’agriculture homogène. La Petite Région Agricole (PRA) est constituée par le croisement du département et de la RA. 

En gros, le découpage administratif classique (région & département) ne permet pas de prendre en compte les caractéristiques météorologique, pédologique et social qui forment l'extraordinaire hétérogénéité de l'agriculture française.  
On a donc crée après la guerre un nouveau découpage prenant en compte au mieux toutes ces caractéristiques. Une sorte de "fork" de la notion de **terroir**, mais mieux définit. 

## Création des shapefile

Pour créer ces fichiers, je suis parti de la liste des communes 2007 fournit par l'INSEE et de la liste de correspondance pra-communes fournit par le ministère de l'agriculture.  
J'ai crée à la main une table complète des petites régions agricoles à partir de la liste originelle et des groupements de petites régions agricoles utilisés pour les statistiques FNSAFER et du ministère.
Quelques formules matricielles Excel et beaucoup de corrections à la main me bouffèrent plusieurs journées pour que la liste des communes comporte une colonne ManyToOne vers la liste des pra.  
Après avoir télécharger le fichier shapefile des communes françaises 2015 par OpenStreetMap au 100m, une jointure a été établi vers le fichier csv des communes. Des petites corrections à la main furent nécessaire à cause des annuelles fusions et séparations de communes française.  
  
Pour terminer, un peu magie postgis :
```sql
CREATE TABLE pra AS
SELECT ST_MULTI(ST_UNION(geom)) :: GEOMETRY(MULTIPOLYGON, 4284) as geom, code_pra, name_pra
FROM communes
GROUP BY communes.code_pra, communes.name_pra
```
et export

voilà

## Contenu du projet

##### /list
Deux fichiers csv
* Liste des communes en 2007 (Corse et DOM-TOM inclus) avec clé pra
* Liste des pra
    * Colonne 1 : Toutes les pra existantes
    * Colonne 2 : Le groupement des pra utilisé par la SAFER
    * Colonne 3 : Le groupement des pra utilisé par le ministère dans les anciennes séries de statistiques
    * Colonne 4 : Les pra selon la liste officielle
  
A noter que :  
1. Il est fort possible qu'il reste quelques coquilles dans la liste des communes concernant leur pra (moins de 50 coquilles sûr)
2. Le liste totales des pra est supérieur à la liste officielle car la SAFER et le Ministère ont divisé certaines pra. Exemple :
    * Pays d'Auge -> Pays d'Auge nord et Pays d'Auge sud
    * Plaine de Caen -> Plaine de Caen nord et Plaine de Caen sud
    * Choletais -> Les Mauges et Le Lavon
    * Vallée de la Sarthe et région mancelle -> Vallée de la Sarthe et région mancelle, Vallée de la Sarthe nord, Vallée de la Sarthe sud et est
3. La liste officielle des PRA était toute en majuscule et comportait pas mal de coquille dans l'orthographe des noms. 
J'ai fait de mon mieux pour corriger l'orthographe mais j'ai été moins discipliné concernant les mots avec majuscule.
    * Est-ce que nord et sud prennent une majuscule ?
    * Est-ce que montagne prend une majuscule ?
    * Ce mot est-il commun ou propre ?
    * Franchement j'étais souvent paumé...

##### /gpra (Groupement de petites régions agricoles)

1. Shapefile des groupement selon la FNSAFER
    * Utilisé sur le site [prix-des-terres](http://www.le-prix-des-terres.fr/)
    * Dans leur publication annuelle
    * Depuis 2016 (il me semble) par l'[agreste](http://agreste.agriculture.gouv.fr/)
2. Shapefile des groupements selon le ministère de l'agriculture (2000-2016 il me semble)
    * Utilisé pour les publications dans le journal officiel des arrêtés sur les valeurs vénales des terres de 2000 à 2015 (2009 à 2010 n'existe pas)
    * L'Agreste a, semble-t-il, choisi le groupement SAFER (avec qui ils font les statistiques de toutes façon) depuis 2016.
    * Les statistiques pré-2000 utilisent plutôt les pra officielles

> Ces groupements répondent mieux à l'évolution de l'agriculture qu'à entrainer les mutations technologiques et économiques.


##### /pra (Petites régions agricoles)

1. Shapefile de toutes les pra 
    * Après découpage du Pays d'Auge, Plaine de Caen, Choletais, Vallée de la Sarthe et région mancelle
2. Shapefile selon la liste officielle  

##### /ra (Régions agricoles)

Crée en faisant un ST_UNION des geom par rapport au nom

1. Shapefile selon toutes les ra
    * Après découpage du Pays d'Auge, Plaine de Caen Choletais, Vallée de la Sarthe et région mancelle
2. Shapefile selon la liste officielle

##### Nombre de géométrie par shapefile

Liste | Features
------|---------
Régions agricoles  | 552
Régions agricoles Insee | 546
Petites Régions agricoles | 723
Petites Régions agricoles Insee | 718
Groupement de petites régions agricoles FNSAFER | 391
Groupement de petites régions agricoles Ministère | 396


## FAQ

**Est-ce que c'était pénible ?**  
Oui    
**Pourquoi alors ?**  
J'en avais besoin pour mon boulot et pour mon site [agrillion.fr](http://www.agrillion.fr)    

## Liste du "restant à faire"  

1. Faire une passe de vérification sur la Gironde et le Lot-et-Garonne
2. Constituer une liste des régions agricoles dans les DOM-TOM utilisées dans les statistiques pour les valeurs des terres
3. Carte des régions viticoles