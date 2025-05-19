# Documentation Wireshark

## Introduction

**Wireshark** est un analyseur de paquets réseau open source permettant de capturer et d’analyser les trames circulant sur un réseau en temps réel. Il prend en charge de nombreux protocoles et propose des outils de filtrage avancés pour faciliter l’analyse du trafic. Ce projet explore les fonctionnalités de Wireshark, les protocoles réseau tels que ARP, UDP et TCP, ainsi que les formats de capture comme PCAP et PCAPNG.

---

## Partie 1 : Analyse de base avec Wireshark

### 1. Différence entre trame et paquet

Dans le domaine des réseaux, une **trame** et un **paquet** sont des unités de données distinctes, correspondant à différentes couches du modèle OSI :

| **Élément**        | **Couche OSI**              | **Contenu**                                                                 |
|--------------------|-----------------------------|-----------------------------------------------------------------------------|
| **Trame (Frame)**  | Liaison de données (Couche 2) | - Adresses MAC source et destination <br> - En-tête Ethernet <br> - Données encapsulées |
| **Paquet (Packet)**| Réseau (Couche 3)           | - Adresses IP source et destination <br> - En-tête IP (TTL, protocole, etc.) |

**Exemple** :
- **Trame** : `34:27:92:45:42:8b → 00:0c:29:fc:e1:ce`
- **Paquet** : `192.168.1.81 → 192.168.1.254`

---

### 2. Formats PCAP vs PCAPNG

Wireshark utilise deux formats principaux pour enregistrer les captures réseau :

| **Format** | **Description**                                      | **Avantages**                              |
|------------|------------------------------------------------------|--------------------------------------------|
| **PCAP**   | Format historique pour les captures réseau           | Compatibilité avec de nombreux outils      |
| **PCAPNG** | Version améliorée, successeur de PCAP                | Supporte les métadonnées (interfaces, timestamps précis, etc.) |

---

### 3. Capture des paquets

#### Configuration initiale

Pour capturer des paquets, Wireshark doit souvent être lancé avec des privilèges administrateurs pour accéder aux interfaces réseau :

```bash
sudo wireshark
```

Sélectionnez ensuite l’interface réseau appropriée (ex. : `eth0`, `ens33`) pour démarrer la capture.

#### Filtres de capture courants

Wireshark permet de filtrer le trafic capturé selon les protocoles :

| **Protocole** | **Filtre** | **Exemple d'utilisation**            |
|---------------|------------|--------------------------------------|
| ARP           | `arp`      | Analyse des résolutions d'adresses   |
| UDP           | `udp`      | Surveillance de flux en temps réel   |
| TCP           | `tcp`      | Suivi des connexions HTTP/FTP        |

#### Exemples de captures

- **Requête ARP** :
  ```plaintext
  FreeboxS_45:42:8b → VMware_fc:el:ce : "Who has 192.168.1.81? Tell 192.168.1.254"
  ```
- **Réponse ARP** :
  ```plaintext
  VMware_fc:el:ce → FreeboxS_45:42:8b : "192.168.1.81 is at 00:0c:29:fc:e1:ce"
  ```

---

### 4. Désencapsulation et modèle OSI

Les trames capturées peuvent être analysées pour observer l’encapsulation des données à travers les couches du modèle OSI. Exemple pour une trame UDP :

```plaintext
Ethernet II (Couche 2) → IP (Couche 3) → UDP (Couche 4) → Données applicatives (Couche 7)
```

**Analyse hexadécimale** d’une trame UDP :

```plaintext
0000   34 27 92 45 42 80 00 0c 29 fc e1 ce 08 00 45 00  # En-tête Ethernet + IP
0010   00 25 c1 c5 40 00 40 11 f4 62 c0 a8 01 51 c0 a8  # En-tête UDP
0020   01 fe e9 34 00 50 00 11 84 c2 54 65 73 74 20 55  # Données ("Test UDP")
```

---

### 5. Tableau d'adresses MAC/IP

Un suivi des adresses MAC et IP est utile lors de l’analyse des trames :

| **Type**       | **MAC Source**          | **IP Source**      | **MAC Destination**     | **IP Destination**   |
|----------------|-------------------------|--------------------|-------------------------|----------------------|
| Requête ARP    | `FreeboxS_45:42:8b`     | `192.168.1.254`    | `VMware_fc:el:ce`       | `192.168.1.81`      |
| Réponse ARP    | `VMware_fc:el:ce`       | `192.168.1.81`     | `FreeboxS_45:42:8b`     | `192.168.1.254`     |
| Paquet UDP     | `VMware_fc:el:ce`       | `192.168.1.81`     | `FreeboxS_45:42:8b`     | `192.168.1.254:80`  |

---

### 6. Protocoles complémentaires

En plus d’ARP, UDP et TCP, Wireshark peut analyser d’autres protocoles comme **ICMP** :

- **ICMP (Internet Control Message Protocol)** :
  - **Fonction** : Vérifie la connectivité réseau (ex. : avec `ping`).
  - **Exemple** :
    ```plaintext
    192.168.1.254 → 192.168.1.81 : Echo (ping) request
    192.168.1.81 → 192.168.1.254 : Echo (ping) reply
    ```

---

### 7. Détails des protocoles

#### ARP (Address Resolution Protocol)

ARP mappe une adresse IP à une adresse MAC dans un réseau local.

**Structure d’une trame ARP** :

| **Champ**             | **Description**                      |
|-----------------------|--------------------------------------|
| MAC Source            | Adresse physique de l’émetteur       |
| IP Source             | Adresse logique de l’émetteur        |
| MAC Destination       | `ff:ff:ff:ff:ff:ff` (broadcast) pour les requêtes |
| IP Destination        | Adresse IP cible                     |
| Opcode                | `1` (Requête) ou `2` (Réponse)       |

**Exemple** :
- **Requête ARP** : `Who has 192.168.1.81? Tell 192.168.1.254`
- **Réponse ARP** : `192.168.1.81 is at 00:0c:29:fc:e1:ce`

#### Three-Way Handshake TCP

Le **Three-Way Handshake** établit une connexion TCP fiable :

| **Étape**         | **Flags**     | **Séquence** | **Accusé** | **Direction**         |
|--------------------|---------------|--------------|------------|-----------------------|
| SYN (Client)      | `SYN`         | 0            | –          | Client → Serveur      |
| SYN-ACK (Serveur) | `SYN, ACK`    | 0            | 1          | Serveur → Client      |
| ACK (Client)      | `ACK`         | 1            | 1          | Client → Serveur      |
| FIN (Serveur)     | `FIN, PSH, ACK` | 421         | 608        | Serveur → Client      |
| FIN (Client)      | `FIN, ACK`    | 608          | 422        | Client → Serveur      |
| ACK Final         | `ACK`         | 422          | 609        | Serveur → Client      |

---

## Partie 2 : Protocoles avancés et sécurité

### 1. Capture de protocoles spécifiques

Wireshark permet d’analyser des protocoles complexes utilisés dans des contextes professionnels :

| **Protocole** | **Filtre** | **Cas d'usage**                     |
|---------------|------------|-------------------------------------|
| DHCP          | `dhcp`     | Attribution d’adresses IP dynamiques|
| DNS           | `dns`      | Résolution de noms de domaine       |
| FTP           | `ftp`      | Transfert de fichiers (non sécurisé)|
| SMB           | `smb`      | Partage de ressources Windows       |

#### Risques de sécurité avec FTP

FTP transmet les données en clair, exposant les identifiants :

**Exemple** :
```plaintext
USER admin      # Hex: 55 53 45 52 20 61 64 6d 69 6e
PASS 1234       # Hex: 50 41 53 53 20 31 32 33 34
```
> **Attention** : Les identifiants sont visibles sans chiffrement.

#### SSL/TLS

Les protocoles comme **SSL/TLS** (HTTPS, FTPS) chiffrent les données, mais Wireshark ne peut pas les déchiffrer sans la clé privée.

---

## Partie 3 : Automatisation avec Tshark

**Tshark**, la version en ligne de commande de Wireshark, permet d’automatiser les captures et analyses.

### 1. Installation

Sur Debian/Ubuntu :
```bash
sudo apt install tshark
```

### 2. Commandes utiles

| **Objectif**          | **Commande**                                      |
|-----------------------|---------------------------------------------------|
| Capture ARP           | `tshark -i <interface> -f "arp"`                 |
| Capture UDP           | `tshark -i <interface> -f "udp"`                 |
| Capture DNS           | `tshark -i <interface> -f "port 53"`             |
| Extraction de métadonnées | `tshark -r capture.pcap -T fields -e ip.src`     |

---
