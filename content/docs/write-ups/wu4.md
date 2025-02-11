---
title: Write-up 4
prev: "/write-ups/wu2"
---


# 🏰 Brute Force sur les Fichiers et Répertoires - Castral-Roc

## 🎯 Objectif
John cherche une entrée cachée pour infiltrer **Castral-Roc**. Il doit **brute-forcer les fichiers et répertoires** sur le sous-domaine **lannister.castral-roc.lannisport.south** afin de découvrir **une page d’authentification secrète**.

**Flag attendu :** high_login.php

---

## ⚙️ Environnement
- **Cible :** lannister.castral-roc.lannisport.south
- **Outil utilisé :** gobuster
- **Wordlist :** /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

---

## 🚀 Étape 1 : Vérifier l’accessibilité du serveur
1. **Tester la résolution DNS :**  
   ```bash
   nslookup lannister.castral-roc.lannisport.south
   ```
2. **Vérifier si le serveur répond :**  
   ```bash
   curl -I http://lannister.castral-roc.lannisport.south
   ```

---

## 🔍 Étape 2 : Scanner les fichiers avec gobuster
Lancer un scan des répertoires pour détecter des fichiers cachés :
```bash
gobuster dir -u http://lannister.castral-roc.lannisport.south \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x php,html,txt
```

**Résultat attendu :**
```
/admin                 (403 Forbidden)
/backup                (403 Forbidden)
/private               (200 OK)
/high_login.php        (200 OK)
```
🎯 **Le flag high_login.php est trouvé !** 🎯

---

## 🚀 Étape 3 : Contourner les protections
Si gobuster ne trouve rien, exclure les erreurs 403 :
```bash
gobuster dir -u http://lannister.castral-roc.lannisport.south \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x php,html,txt -b 403
```

Si le serveur bloque encore le scan, tester avec un **User-Agent différent** :
```bash
gobuster dir -u http://lannister.castral-roc.lannisport.south \
-w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x php,html,txt \
-a "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
```

---

## 🎯 Étape 4 : Validation du Flag
Une fois high_login.php trouvé, confirmer en accédant à :
```
http://lannister.castral-roc.lannisport.south/high_login.php
```
**Flag final :** high_login.php

---

## 🔒 Mitigation & Sécurisation
- **Désactiver l’indexation des fichiers (autoindex off;).**
- **Restreindre l’accès avec un .htaccess ou une configuration Nginx.**
- **Utiliser fail2ban pour bloquer les scans abusifs.**

---

## 🏆 Conclusion
- **gobuster a révélé une page cachée (high_login.php).**
- **L’attaque est réussie, l’entrée vers Castral-Roc est trouvée !** 🔥