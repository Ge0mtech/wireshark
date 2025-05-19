# Documentation Wireshark 

## Introduction

**Wireshark** est un outil gratuit qui permet de regarder ce qui se passe dans un réseau informatique en temps réel. Imaginez-le comme une loupe qui vous aide à voir les messages (ou "paquets") envoyés entre les ordinateurs, les routeurs ou tout autre appareil connecté. Il est utile pour repérer des problèmes réseau, apprendre comment les réseaux fonctionnent ou analyser le trafic.

---

## Partie 1 : Analyse de base avec Wireshark

### 1. Différence entre trame et paquet

Quand des données circulent dans un réseau, elles sont divisées en petites unités appelées **trames** et **paquets**. Voici la différence, expliquée simplement :

- **Trame** : C’est comme une enveloppe qui contient les données. Elle indique les adresses physiques (MAC) de l’appareil qui envoie et de celui qui reçoit. Elle fonctionne au niveau "local", comme un facteur qui livre dans votre quartier.
- **Paquet** : C’est ce qu’il y a dans l’enveloppe, avec les adresses IP (comme une adresse postale) qui montrent d’où viennent les données et où elles vont.

**Exemple** :  
- Une trame dit : "De l’appareil A à l’appareil B" (avec leurs adresses MAC).  
- Un paquet dit : "De 192.168.1.81 à 192.168.1.254" (avec leurs adresses IP).

---

### 2. Formats PCAP et PCAPNG

Wireshark enregistre ce qu’il voit dans deux types de fichiers :

- **PCAP** : L’ancien format, simple et compatible avec presque tous les outils d’analyse réseau.
- **PCAPNG** : Une version plus moderne qui peut stocker plus de détails, comme l’heure exacte ou des infos sur votre connexion.

**En bref** : PCAP est basique et universel, PCAPNG est plus riche en informations.

---

### 3. Capture des paquets

Pour voir le trafic réseau avec Wireshark, suivez ces étapes simples :

1. Ouvrez Wireshark (parfois, il faut être administrateur sur votre ordinateur).
2. Choisissez la connexion à surveiller (par exemple, Wi-Fi ou câble Ethernet).
3. Cliquez sur "Démarrer" pour commencer à capturer.

#### Filtres pour simplifier

Vous pouvez choisir ce que vous voulez voir :

- **ARP** : Pour repérer qui demande des adresses dans le réseau.
- **UDP** : Pour suivre des données rapides, comme une vidéo en streaming.
- **TCP** : Pour voir des connexions stables, comme un site web ou un téléchargement.

#### Exemples concrets

- **Demande ARP** : Un appareil dit "Qui a l’adresse IP 192.168.1.81 ? Répondez à 192.168.1.254 !".
- **Réponse ARP** : Un autre répond "C’est moi, et voici mon adresse MAC !".

---

### 4. Désencapsulation et modèle OSI

Les données dans un réseau sont comme des poupées russes : elles sont emballées dans plusieurs couches. Wireshark vous permet de les "déballer" pour voir :

- **Couche 2** : Les adresses MAC (l’enveloppe).
- **Couche 3** : Les adresses IP (l’adresse sur l’enveloppe).
- **Couche 4** : Comment les données voyagent (TCP ou UDP, comme le mode d’envoi).
- **Couche 7** : Le contenu réel (comme une lettre ou une vidéo).

En regardant une trame, vous pouvez voir toutes ces couches s’emboîter.

---

### 5. Tableau d’adresses MAC/IP

Voici un tableau simple pour suivre qui parle à qui dans une capture :

| **Type**       | **Adresse MAC émetteur** | **IP émetteur** | **Adresse MAC destinataire** | **IP destinataire** |
|----------------|--------------------------|-----------------|-----------------------------|---------------------|
| Demande ARP    | `FreeboxS_45:42:8b`      | `192.168.1.254` | `VMware_fc:el:ce`           | `192.168.1.81`      |
| Réponse ARP    | `VMware_fc:el:ce`        | `192.168.1.81`  | `FreeboxS_45:42:8b`         | `192.168.1.254`     |
| Paquet UDP     | `VMware_fc:el:ce`        | `192.168.1.81`  | `FreeboxS_45:42:8b`         | `192.168.1.254:80`  |

Chaque ligne montre une "conversation" entre deux appareils.

---

### 6. Protocoles complémentaires

Wireshark peut aussi voir d’autres protocoles, comme **ICMP** :

- **ICMP** : C’est comme un test pour vérifier si un appareil répond. Par exemple, quand vous utilisez `ping`, ICMP envoie une question ("Es-tu là ?") et attend une réponse ("Oui, je suis là !").

---

### 7. Détails des protocoles

#### ARP (Address Resolution Protocol)

ARP aide les appareils à se trouver dans un réseau local, comme un annuaire qui donne l’adresse physique (MAC) à partir d’une adresse IP.

**Exemple** :  
- Demande : "Qui a 192.168.1.81 ? Dites-le à 192.168.1.254 !"  
- Réponse : "C’est moi, et mon adresse MAC est 00:0c:29:fc:e1:ce."

#### Three-Way Handshake TCP

TCP utilise une "poignée de main" en trois étapes pour démarrer une connexion fiable (par exemple, pour charger une page web) :

1. Le client dit : "Salut, je veux me connecter !"  
2. Le serveur répond : "OK, connectons-nous !"  
3. Le client confirme : "Parfait, c’est parti !"

Une fois ces étapes terminées, les données peuvent circuler sans problème.

---

## Partie 2 : Protocoles avancés et sécurité

### 1. Capture de protocoles spécifiques

Wireshark peut analyser des protocoles plus complexes :

- **DHCP** : Donne automatiquement une adresse IP aux appareils (comme un employé qui attribue un bureau).
- **DNS** : Transforme un nom comme "google.com" en adresse IP (comme un annuaire pour sites web).
- **FTP** : Envoie des fichiers, mais sans protection.
- **SMB** : Partage des fichiers ou imprimantes dans un réseau Windows.

#### Risques avec FTP

FTP n’est pas sécurisé : il envoie les mots de passe en clair. Par exemple, si vous entrez "admin" et "1234", quelqu’un peut les voir avec Wireshark. C’est comme écrire votre mot de passe sur une affiche !

**Solution** : Utilisez **SFTP** ou **FTPS**, qui protègent vos données.

#### SSL/TLS

**SSL/TLS** (comme dans HTTPS) rend les données illisibles pour ceux qui n’ont pas la clé secrète. Wireshark peut voir qu’il y a du trafic, mais pas ce qu’il contient.

---

## Partie 3 : Automatisation avec Tshark

**Tshark** est une version de Wireshark qui fonctionne sans interface graphique, idéale pour automatiser des tâches.

### 1. Installation

Sur Linux (comme Ubuntu), installez-le avec :  
```bash
sudo apt install tshark
```

### 2. Commandes simples

- Voir les demandes ARP : `tshark -i <interface> -f "arp"`  
- Capturer du trafic UDP : `tshark -i <interface> -f "udp"`  
- Surveiller le DNS : `tshark -i <interface> -f "port 53"`

Tshark est parfait pour des analyses rapides ou programmées.
