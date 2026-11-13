# reseaux-packet-tracer
# 03 – Protection multicouche des ports d'accès

> ⚠️ **À compléter par moi avant de publier** : les passages entre crochets `[...]` (captures, résultats observés). Ne rien laisser que je n'ai pas réellement fait ou vérifié.

## Objectif
Sécuriser un même VLAN contre quatre comportements non autorisés en combinant **Port Security**, **DHCP Snooping**, **IP Source Guard** et **Dynamic ARP Inspection (DAI)**, puis valider la protection d'ensemble.

## Topologie
Reprise du laboratoire 02, avec `PC-TEST` ajouté sur `Fa0/4` pour les tests d'usurpation d'adresse IP.

| Équipement | Port S1 | Rôle |
|------------|---------|------|
| PC-A | Fa0/1 | Client légitime |
| PC-B | Fa0/2 | Client / essai de MAC |
| S-ROGUE | Fa0/3 | Faux serveur DHCP |
| PC-TEST | Fa0/4 | Test IP Source Guard / DAI |
| R1 (Gi0/0) | Fa0/24 | Chemin DHCP et passerelle (port trusted) |

Adressage : VLAN 10 `192.168.10.0/24` (DHCP `100-199`), `R1` `192.168.10.1`, `PC-EXT` `172.16.20.10`.

![Topologie](captures/03_topologie.png)
<img width="619" height="354" alt="image" src="https://github.com/user-attachments/assets/ed59aa7b-cea0-4f9f-bd34-194062904319" />


## Protections et menaces visées
| Protection | Menace | Indicateur observé |
|------------|--------|--------------------|
| Port Security | MAC non autorisée sur un port | MAC sécurisées, violations |
| DHCP Snooping | Faux serveur DHCP | Ports trusted, table de bindings |
| IP Source Guard | Paquets avec IP source usurpée | `show ip verify source` |
| DAI | Messages ARP mensongers (ARP spoofing) | Compteurs de rejets ARP |

## Démarche (une protection à la fois, en observant l'effet)
- **Phase A** – Port Security sur `Fa0/1`
- **Phase B** – DHCP Snooping (`Fa0/24` trusted)
- **Phase C** – IP Source Guard sur `Fa0/4` (utilise les bindings du DHCP Snooping)
- **Phase D** – DAI sur le VLAN 10 (`Fa0/24` trusted, limitation du débit ARP)
- **Validation intégrée** et analyse des limites

## Configuration clé
```
! Phase B
ip dhcp snooping
ip dhcp snooping vlan 10
interface fastEthernet 0/24
 ip dhcp snooping trust

! Phase C
interface fastEthernet 0/4
 ip verify source

! Phase D
ip arp inspection vlan 10
interface fastEthernet 0/24
 ip arp inspection trust
interface fastEthernet 0/1
 ip arp inspection limit rate 15
```

## Vérifications
```
show port-security interface fastEthernet 0/1
show ip dhcp snooping binding
show ip verify source
show ip source binding
show ip arp inspection
```
[Ajouter les captures de chaque phase, avant / après.]

## Résultats
| Test | Attendu | Observé |
|------|---------|---------|
| MAC non autorisée sur Fa0/1 | Violation | [à compléter] |
| Faux serveur DHCP sur Fa0/3 | Offres bloquées | [à compléter] |
| IP source usurpée depuis Fa0/4 | Paquets rejetés | [à compléter] |
| Annonce ARP mensongère | Paquets ARP rejetés | [à compléter] |

## Ce que j'ai retenu
- Les protections se **complètent** : le DHCP Snooping fournit les bindings qui alimentent IP Source Guard et DAI.
- Il faut déclarer `trust` sur le port vers le routeur, sinon les réponses légitimes sont bloquées.
- L'ordre d'activation compte : activer DAI / IP Source Guard avant que les bindings existent coupe les clients légitimes.

📁 Fichier : `03_Protection_Multicouche.pkt`

[TP03.docx](https://github.com/user-attachments/files/34819774/TP03.docx)

