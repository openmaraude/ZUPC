= Fusion de communes

Chaque année début janvier un utilisateur d'OpenStreetMap publie un dump du territoire des communes françaises
qui nous sert à détecter si un taxi est bien présent dans sa zone autorisée pour le rendre visible.

Nous avons une commande flask `import_towns` qui va télécharger et déployer les nouvelles données.

Mais chaque année il y a des fusions de communes, c'est à dire des petites communes (généralement)
qui se regroupent ou sont absorbées, et transfèrent leur pouvoir à une commune unique qui adopte
généralement un nouveau nom. La nouvelle entité administrative est qualifiée de commune nouvelle,
et les anciennes de communes déléguées.

Ces communes déléguées seront supprimées car le dump ne contient que les communes principales,
pas le détail des communes déléguées et autres lieux-dits. Ces communes existent toujours,
on croisera encore le panneau de la route à l'entrée de la commune, mais elles n'ont plus
le pouvoir de police de délivrer des autorisations de stationnement.

Exemple de sortie dans l'environnement de développement avec l'import 2022 :

```
[2022-01-04 14:40:32,049] INFO in towns: Inserting new town Les Trois Lacs (27058)
[2022-01-04 14:44:25,928] INFO in towns: Deleting old town Châtillon-sur-Lison (25134)
[2022-01-04 14:44:25,928] INFO in towns: Deleting old town La Faute-sur-Mer (85307)
[2022-01-04 14:44:25,928] INFO in towns: Deleting old town Orliaguet (24314)
[2022-01-04 14:44:25,928] INFO in towns: Deleting old town Ambleville (16010)
[2022-01-04 14:44:25,929] INFO in towns: Deleting old town Cazoulès (24089)
[2022-01-04 14:44:25,929] INFO in towns: Deleting old town Mureils (26219)
[2022-01-04 14:44:25,929] INFO in towns: Deleting old town Croixanvec (56049)
[2022-01-04 14:44:25,929] INFO in towns: Deleting old town Saint-Barthélemy (97701)
[2022-01-04 14:44:25,930] INFO in towns: Deleting old town Les Trois Lacs (27676)
[2022-01-04 14:44:25,930] INFO in towns: Deleting old town Le Jardin (19092)
[2022-01-04 14:44:25,930] INFO in towns: Deleting old town Saint-Pierre (97502)
[2022-01-04 14:44:25,930] INFO in towns: Deleting old town Miquelon-Langlade (97501)
[2022-01-04 14:44:25,930] INFO in towns: Deleting old town Saint-Thibaut (02695)
[2022-01-04 14:44:25,931] INFO in towns: Deleting old town Saint-Martin (France) (97801)
Done
```

On ne peut pas se permettre de supprimer une commune si des taxis y sont enregistrés.

Il nous faut donc créer un mapping pour les "migrer" vers la commune nouvelle.
C'est le rôle des fichiers YAML de ce répertoire.

Les éventuelles ADS utilisant les codes INSEE en clé du mapping seront migrées
au nouveau code INSEE associé à chaque clé.

Le code INSEE de la commune nouvelle est simplement trouvé en partant
de la page Wikipédia de l'ancienne commune, il est toujours annoncé que c'est
une commune déléguée, avec un lien vers la page de la commune nouvelle
où l'on trouve le "code commune" dans la colonne de droite.

La commune nouvelle réutilise le code INSEE de l'une des communes fusionnées,
généralement la plus grande, donc cette dernière n'a pas besoin de figurer ici.
On met juste à jour le nom, c'est sans conséquence sur notre modèle de données.

Si des ZUPC de ce repo utilisent l'un des anciens codes INSEE, il faut mettre à jour
leur déclaration (`git grep -E 'X|Y|Z'` pour les trouver).

Toujours commencer par tester la commande import_towns avec un dump de prod
sur son environnement de développement.

L'import 2022 est particulièrement intéressant car il contient je pense tous les cas
possibles. J'ai documenté les changements dans le fichier `2022.yaml` lui-même.
