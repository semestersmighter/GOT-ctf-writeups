---
title: Write-up 1
next: "/write-ups/wu2"
prev: ""
---

# 🗂️ Exposition d’informations sensibles CWE-200

## 🎯 Objectif

Pour pouvoir atteindre Lannisport, John va devoir trouver une carte (sous forme d’une URL) et devra être accompagné de son loup pour traverser le pays sans encombre. 
Le challenge consiste donc à le trouver sur le site de la maison Stark

**Flag attendu :** l’URL et le nom du loup

---

##  Environnement
- **Prérequis :** rien

---

## 🚀 Étape 1 : Recherche du nom du loup :

```bash
    wget https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Discovery/Web-Content/common.txt -O common_short.txt
    gobuster dir -u http://stark.north/ -w common_short.txt
```


On go buster sur le site stark.north
On trouve un robots.txt

---

## Étape 2 : Identifier les colonnes visibles
on va dessus du coup
http://ontheway.travel/castral-roc

---

## Étape 3 : Déterminer le nom des tables de la bdd 
http://stark.north/
On traine sur dans le inspecter et on trouve dans audio.js: 
//J'aime bien écouter cette musique en compagnie de mon loup qui s'appelle wolfy

---

## Étape 4 : Déterminer le nom des colonnes de la table battle_ressources 

http://ontheway.travel/castral-roc?loup=wolfy

---

