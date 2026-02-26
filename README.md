# Configuration du Routage Dynamique RIPv2

## Description
Ce dépôt contient les fichiers et la documentation pour le TP 10 sur la configuration du routage dynamique avec le protocole **RIPv2** sur des équipements Cisco via Packet Tracer.

L'objectif de ce TP est de :
1. Appliquer les paramètres de base d'un routeur et d'un commutateur.
2. Configurer les interfaces et le plan d'adressage IP.
3. Configurer le routage dynamique avec le protocole RIP version 2.
4. Analyser les tables de routage et tester la connectivité.

## Topologie
La topologie comprend trois routeurs (R1, R2, R3) interconnectés, deux commutateurs (S1, S2) et quatre ordinateurs (PC1, PC2, PC3, PC4). 
*Note : Les noms des interfaces peuvent varier selon le modèle de routeur utilisé dans Packet Tracer (ex: `Gig0/0` vs `Gig0/0/0`).*

## Configuration des équipements (Scripts complets)

### 1. Configuration des Commutateurs (Switchs)

#### Switch S1
```text
enable
configure terminal
hostname S1
no ip domain-lookup
no cdp run
interface vlan 1
 ip address 192.168.0.2 255.255.255.0
 no shutdown
end
copy running-config startup-config
```

#### Switch S2
```text
enable
configure terminal
hostname S2
no ip domain-lookup
no cdp run
interface vlan 1
 ip address 10.0.0.2 255.255.255.0
 no shutdown
end
copy running-config startup-config
```

### 2. Configuration des Routeurs (Adresses IP + Routage RIPv2)

#### Routeur R1 (Router0 sur la maquette)
```text
enable
configure terminal
hostname R1
no ip domain-lookup
no cdp run

! Configuration des interfaces
interface Gig0/0/0
 description Lien vers le Switch S1
 ip address 192.168.0.1 255.255.255.0
 no shutdown

interface Gig0/0/1
 description Lien P2P vers Router1
 ip address 209.165.200.225 255.255.255.252
 no shutdown

! Configuration du routage dynamique RIP version 2
router rip
 version 2
 no auto-summary
 network 192.168.0.0
 network 209.165.200.224
 passive-interface Gig0/0/0

end
copy running-config startup-config
```

#### Routeur R2 (Router1 sur la maquette)
```text
enable
configure terminal
hostname R2
no ip domain-lookup
no cdp run

! Configuration des interfaces
interface Gig0/0/0
 description Lien P2P venant de Router0
 ip address 209.165.200.226 255.255.255.252
 no shutdown

interface Gig0/0/1
 description Lien P2P vers Router2
 ip address 209.65.100.225 255.255.255.252
 no shutdown

! Configuration du routage dynamique RIP version 2
router rip
 version 2
 no auto-summary
 network 209.165.200.224
 network 209.65.100.224

end
copy running-config startup-config
```

#### Routeur R3 (Router2 sur la maquette)
```text
enable
configure terminal
hostname R3
no ip domain-lookup
no cdp run

! Configuration des interfaces
interface Gig0/0/1
 description Lien vers le Switch S2
 ip address 10.0.0.1 255.255.255.0
 no shutdown

interface Gig0/0/0
 description Lien P2P venant de Router1
 ip address 209.65.100.226 255.255.255.252
 no shutdown

! Configuration du routage dynamique RIP version 2
router rip
 version 2
 no auto-summary
 network 209.65.100.224
 network 10.0.0.0
 passive-interface Gig0/0/1

end
copy running-config startup-config
```

---

### 5. Vérifications et Tests de connectivité

#### Affichage de la table de routage sur R1
Pour s'assurer que le protocole RIP fonctionne, nous affichons la table de routage sur le routeur R1 avec la commande :
```text
show ip route
```

**Analyse des résultats attendus :**
- **Types de routes présentes** : On doit observer des routes locales (`L`), des réseaux directement connectés (`C`) et surtout des routes apprises dynamiquement via RIP (`R`).
- **Routes connectées (`C`)** : Il y en a au moins 2 sur R1. Elles représentent les réseaux physiquement branchés au routeur (le LAN 192.168.0.0/24 et le lien P2P 209.165.200.224/30).
- **Routes apprises dynamiquement (`R`)** : Il doit y en avoir 2 sur R1. Elles représentent les réseaux distants que R1 a appris grâce à RIP (le réseau P2P entre R2 et R3, et le LAN de destination 10.0.0.0/24).

#### Test de connectivité
Depuis PC1 (192.168.0.10), on lance un ping vers PC4 (10.0.0.11) :
```text
ping 10.0.0.11
```
*(Dans la simulation, le premier ping perd souvent un ou deux paquets à cause de la résolution ARP sur le réseau, c'est normal. Les pings suivants doivent tous réussir).*

**Conclusion :**
Tous les périphériques répondent correctement car le protocole de routage dynamique RIPv2 a permis à Chaque routeur de partager sa table de connaissance avec ses voisins. Ainsi, R1 sait désormais par quelle interface il doit faire transiter un paquet destiné au réseau `10.0.0.0/24`.

---
*Réalisé avec Cisco Packet Tracer.*
