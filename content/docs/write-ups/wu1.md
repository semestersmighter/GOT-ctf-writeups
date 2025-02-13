---
title: 1 - Exposition d’informations sensibles
next: "/write-ups/wu2"
---

# 🗂️ Exposition d’informations sensibles CWE-200

## 🎯 Objectif
Pour atteindre **Lannisport**, John doit trouver une **carte** (sous forme d’une **URL**) et être accompagné de son **loup** pour traverser le pays en toute sécurité.  
L’objectif du challenge est donc de découvrir ces informations sur le site de la **maison Stark**.

**🛡️ Flag attendu :**  
- L’**URL**  
- Le **nom du loup**  

---

## ⚙️ Environnement  
- **Prérequis :** Aucun  

---

## 🚀 Étape 1 : Recherche du nom du loup  

Nous commençons par récupérer une **liste de répertoires courants** pour explorer le site Stark :  

```
wget https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Discovery/Web-Content/common.txt -O common_short.txt 
```

Ensuite, nous lançons un scan de répertoires avec Gobuster :
```
gobuster dir -u http://stark.north/ -w common_short.txt
```
👉 Résultat : Nous trouvons un fichier **robots.txt**.

---

## 🔎 Étape 2 : Identifier les colonnes visibles  

Nous explorons l’URL suivante pour identifier des éléments exploitables :
```
http://ontheway.travel/castral-roc
```
---

## 🛠️ Étape 3 : Déterminer le nom des tables de la base de données

En parcourant le site http://stark.north/ et en inspectant le code source (Inspecter l'élément), nous trouvons un indice dans le fichier audio.js :
```
// J'aime bien écouter cette musique en compagnie de mon loup qui s'appelle wolfy
```

---

## 📊 Étape 4 : Déterminer le nom des colonnes de la table battle_ressources

Nous utilisons l’URL suivante avec le nom du loup pour obtenir davantage d’informations sur la base de données :
```
http://ontheway.travel/castral-roc?loup=wolfy
```

---

🎯 **Objectif atteint !** Nous avons récupéré :  
✅ **L’URL**  
✅ **Le nom du loup**