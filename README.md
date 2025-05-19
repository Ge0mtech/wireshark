# Documentation Wireshark  

## Introduction  
Wireshark est un analyseur réseau open source permettant de capturer et d’analyser les trames circulant sur un réseau. Il supporte de nombreux protocoles et offre des outils de filtrage avancés. Ce projet explore les fonctionnalités de Wireshark, les protocoles réseau (ARP, UDP, TCP) et les formats de capture (PCAP/PCAPNG).

---

## Partie 1 : Analyse de base avec Wireshark  

### 1. Différence entre trame et paquet  
| **Élément**      | **Couche OSI**       | **Contenu**                                                                 |
|-------------------|----------------------|-----------------------------------------------------------------------------|
| **Trame (Frame)** | Liaison de données (2) | - Adresses MAC source/destination <br> - En-tête Ethernet <br> - Données encapsulées |
| **Paquet (Packet)** | Réseau (3)          | - Adresses IP source/destination <br> - En-tête IP (TTL, protocole, etc.)           |

**Exemple** :  
- Trame : `34:27:92:45:42:8b → 00:0c:29:fc:e1:ce`  
- Paquet : `192.168.1.81 → 192.168.1.254`  

---

### 2. Formats PCAP vs PCAPNG  
| **Format** | **Description**                                      | **Avantages**                              |
|------------|------------------------------------------------------|--------------------------------------------|
| PCAP      | Format historique pour les captures réseau           | Compatibilité large                        |
| PCAPNG    | Version améliorée (successeur de PCAP)               | Supporte les métadonnées (interfaces, etc.) |

---

### 3. Capture des paquets  
#### Configuration initiale  
1. **Lancement en mode root** :  
   ```bash
   sudo wireshark
   ```  
2. **Sélection de l'interface** :  
   - Choisir l'interface connectée au LAN (ex: `eth0`, `ens33`).  

#### Filtres de capture courants  
| **Protocole** | **Filtre** | **Exemple d'utilisation**            |
|---------------|------------|--------------------------------------|
| ARP           | `arp`      | Analyse des résolutions d'adresses   |
| UDP           | `udp`      | Surveillance de flux temps réel      |
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
**Structure d'une trame UDP** :  
```plaintext
Ethernet II (Couche 2) → IP (Couche 3) → UDP (Couche 4) → Données applicatives (Couche 7)
```

**Analyse hexadécimale** :  
```plaintext
0000   34 27 92 45 42 80 00 0c 29 fc e1 ce 08 00 45 00  # En-tête Ethernet + IP
0010   00 25 c1 c5 40 00 40 11 f4 62 c0 a8 01 51 c0 a8  # En-tête UDP
0020   01 fe e9 34 00 50 00 11 84 c2 54 65 73 74 20 55  # Données ("Test UDP")
```

---

### 5. Tableau d'adresses MAC/IP  
| **Type**       | **MAC Source**          | **IP Source**      | **MAC Destination**     | **IP Destination**   |
|----------------|-------------------------|--------------------|-------------------------|----------------------|
| Requête ARP    | `FreeboxS_45:42:8b`     | `192.168.1.254`    | `VMware_fc:el:ce`       | `192.168.1.81`      |
| Réponse ARP    | `VMware_fc:el:ce`       | `192.168.1.81`     | `FreeboxS_45:42:8b`     | `192.168.1.254`     |
| Paquet UDP     | `VMware_fc:el:ce`       | `192.168.1.81`     | `FreeboxS_45:42:8b`     | `192.168.1.254:80`  |

---

### 6. Protocoles complémentaires  
- **ICMP** :  
  - **Fonction** : Vérification de connectivité (ex: `ping`).  
  - **Exemple** :  
    ```plaintext
    192.168.1.254 → 192.168.1.81 : Echo (ping) request
    192.168.1.81 → 192.168.1.254 : Echo (ping) reply
    ```  

---

### 7. Détails des protocoles  
#### ARP (Address Resolution Protocol)  
**Structure d'une trame ARP** :  
| Champ                 | Description                          |
|-----------------------|--------------------------------------|
| MAC Source            | Adresse physique de l'émetteur       |
| IP Source             | Adresse logique de l'émetteur        |
| MAC Destination       | `ff:ff:ff:ff:ff:ff` (broadcast)     |
| IP Destination        | Adresse IP cible                     |
| Opcode                | `1` (Requête) ou `2` (Réponse)       |

**Exemple** :  
```plaintext
Who has 192.168.1.81? Tell 192.168.1.254  # Requête ARP
192.168.1.81 is at 00:0c:29:fc:e1:ce      # Réponse ARP
```

#### Three-Way Handshake TCP  
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
| **Protocole** | **Filtre** | **Cas d'usage**                     |
|---------------|------------|-------------------------------------|
| DHCP          | `dhcp`     | Attribution d'adresses IP           |
| DNS           | `dns`      | Résolution de noms de domaine       |
| FTP           | `ftp`      | Transfert de fichiers (non sécurisé)|
| SMB           | `smb`      | Partage de ressources Windows       |

#### Risques de sécurité avec FTP  
- **Données en clair** :  
  ```plaintext
  USER admin      # Hex: 55 53 45 52 20 61 64 6d 69 6e
  PASS 1234       # Hex: 50 41 53 53 20 31 32 33 34
  ```  
  > **Attention** : Les identifiants sont visibles sans chiffrement.

#### SSL/TLS  
- **Protection** : Chiffrement des données (ex: HTTPS, FTPS).  
- **Limitation** : Impossible de déchiffrer sans clé privée.  

---

## Partie 3 : Automatisation avec Tshark  

### 1. Installation  
```bash
sudo apt install tshark  # Sur Debian/Ubuntu
```

### 2. Commandes utiles  
| **Objectif**          | **Commande**                                      |
|-----------------------|---------------------------------------------------|
| Capture ARP           | `tshark -i eth0 -Y "arp" -w arp_capture.pcap`    |
| Capture UDP           | `tshark -i eth0 -Y "udp" -w udp_capture.pcap`    |
| Capture DNS           | `tshark -i eth0 -Y "dns" -w dns_capture.pcap`    |
| Extraction de métadonnées | `tshark -r capture.pcap -T fields -e ip.src`    |
