# Projet Dana — Groupe 9

**M1 ATAL — Nantes Université | 2026–2027**

## Objectif

Importer notre jeu de données **CCFr (bibliothèques françaises)**, déjà converti au format **RDF/Turtle**, dans **Apache Jena Fuseki**, puis exécuter et documenter deux requêtes SPARQL.

> La conversion en Turtle est déjà réalisée. Aucune transformation XML ni script Python n'est nécessaire pour suivre ce guide.

## 1. Fichier nécessaire

```text
ProjetDana/
├── README.md
├── data/
│   └── ccfr_full.ttl
│   └── CCFR_INSTITUTION_20260702.xml
└── queries/
    ├── bibliotheques_regions.rq
    └── services_bibliotheques.rq
```


## 2. Démarrer Fuseki

Dans le dossier où se trouve `fuseki-server.jar`, ouvrir un terminal et exécuter :

```powershell
java -jar .\fuseki-server.jar
```

> La commande suppose que la distribution de Fuseki utilisée fournit ce JAR exécutable. Sinon, employer le lanceur fourni avec votre installation.

Ouvrir **http://localhost:3030/** dans un navigateur. Garder le serveur en cours d'exécution.

## 3. Créer le dataset

1. Dans Fuseki, cliquer sur **Manage datasets**.
2. Cliquer sur **Add new dataset**.
3. Saisir le nom : `ccfr-bibliotheques`.
4. Choisir **Persistent (TDB2)** pour conserver les données sur disque.
5. Cliquer sur **Create dataset**.
6. Vérifier que `ccfr-bibliotheques` apparaît dans la liste.

## 4. Importer le fichier Turtle

1. Ouvrir le dataset **`ccfr-bibliotheques`**.
2. Aller dans **Add data** (ou **Upload data**, selon l'interface).
3. Choisir le **Default graph**.
4. Sélectionner `data/ccfr_full.ttl`.
5. Cliquer sur **Upload** et attendre la confirmation.

**Important :** importer le fichier `.ttl`. Pour recommencer avec un dataset vierge, supprimer les données existantes ou recréer le dataset.

## 5. Vérifier l'importation avec SPARQL

Dans le dataset, ouvrir l'onglet **Query**, coller chaque requête et cliquer sur **Run**.

### 5.1 Compter les triplets

```sparql
SELECT (COUNT(*) AS ?nombreTriplets)
WHERE {
  ?s ?p ?o .
}
```

Le résultat doit être supérieur à zéro.

### 5.2 Afficher les premiers triplets

```sparql
SELECT ?s ?p ?o
WHERE {
  ?s ?p ?o .
}
LIMIT 20
```

- `?s` : sujet RDF ;
- `?p` : propriété (prédicat) ;
- `?o` : objet (valeur ou ressource).

### 5.3 Identifier les classes présentes

```sparql
SELECT ?type (COUNT(DISTINCT ?ressource) AS ?nombre)
WHERE {
  ?ressource a ?type .
}
GROUP BY ?type
ORDER BY DESC(?nombre)
```

Cette vérification permet d'identifier les types réellement présents avant d'écrire les requêtes d'analyse.

## 6. Deux requêtes SPARQL d'analyse

> **À vérifier :** les requêtes ci-dessous supposent que le fichier Turtle utilise les classes et propriétés `https://schema.org/` indiquées. Si le résultat est vide, consulter les triplets affichés à l'étape 5 et adapter les chemins de propriétés au modèle RDF réellement importé.

### Requête 1 — Bibliothèques par région

**Objectif :** compter les bibliothèques par région et afficher les 15 régions les plus représentées.

À enregistrer dans `queries/bibliotheques_regions.rq` :

```sparql
PREFIX schema: <https://schema.org/>

SELECT ?region (COUNT(DISTINCT ?bibliotheque) AS ?nombre)
WHERE {
  ?bibliotheque a schema:Library ;
                schema:address ?adresse .
  ?adresse schema:addressRegion ?region .
}
GROUP BY ?region
ORDER BY DESC(?nombre)
LIMIT 15
```

### Requête 2 — Services des bibliothèques

**Objectif :** compter les bibliothèques associées à chaque service renseigné.

À enregistrer dans `queries/services_bibliotheques.rq` :

```sparql
PREFIX schema: <https://schema.org/>

SELECT ?service (COUNT(DISTINCT ?bibliotheque) AS ?nombre)
WHERE {
  ?bibliotheque a schema:Library ;
                schema:availableService ?serviceURI .
  ?serviceURI schema:name ?service .
}
GROUP BY ?service
ORDER BY DESC(?nombre)
LIMIT 15
```

**Note :** si les services sont représentés directement par des littéraux plutôt que des ressources, la seconde requête devra être adaptée.

## 7. Captures et livrables de l'étape 1

- [ ] Capture du dataset `ccfr-bibliotheques` dans Fuseki.
- [ ] Capture confirmant l'importation du fichier Turtle.
- [ ] Résultat du comptage des triplets.
- [ ] Résultat de la requête sur les régions.
- [ ] Résultat de la requête sur les services.
- [ ] Fichiers `.rq` enregistrés dans `queries/`.
- [ ] Brève explication du modèle RDF et des résultats pour la présentation.

## 8. Dépannage rapide

| Problème | Vérification |
|---|---|
| `localhost:3030` inaccessible | Vérifier que le serveur Fuseki est démarré et que le terminal reste ouvert. |
| Dataset absent | Revenir dans **Manage datasets** et vérifier sa création. |
| Import `.ttl` en échec | Vérifier la syntaxe Turtle et consulter le message d'erreur de Fuseki. |
| Aucun triplet | Vérifier le graphe de destination et la confirmation de l'import. |
| Requête d'analyse vide | Inspecter les prédicats et classes réellement présents avec les requêtes de l'étape 5. |
| Données absentes après redémarrage | Vérifier le mode **Persistent (TDB2)** et le répertoire de stockage utilisé. |

---

**Projet Dana — Groupe 9 — M1 ATAL — Nantes Université**
