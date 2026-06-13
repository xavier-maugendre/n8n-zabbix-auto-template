
# 🔄 Auto-Template Zabbix via SNMP OID

> Détection automatique du vendor SNMP → Assignation du bon template Zabbix. Zéro clic, zéro erreur humaine.

![n8n](https://img.shields.io/badge/n8n-workflow-EA4B71?style=flat&logo=n8n&logoColor=white)
![Zabbix](https://img.shields.io/badge/Zabbix-monitoring-CC0000?style=flat)
![SNMP](https://img.shields.io/badge/SNMP-auto--detection-blue?style=flat)
![License](https://img.shields.io/badge/license-MIT-green?style=flat)

---

## Description

Ce workflow n8n détecte automatiquement le type d'équipement réseau via son **System Object ID (OID) SNMP** et assigne le template Zabbix correspondant, en remplaçant le template générique par défaut.

**Le problème résolu :** Quand on intègre un nouvel équipement SNMP dans Zabbix, il hérite d'un template générique. Il faut ensuite manuellement identifier le vendor et changer le template. Sur un parc de centaines d'équipements, c'est des heures de travail. Ce workflow le fait en quelques secondes.

---
<img width="1536" height="1024" alt="ChatGPT Image 13 juin 2026, 12_17_21" src="https://github.com/user-attachments/assets/57dd3e5e-6160-43f3-936b-e0431912ad7f" />

---
## Flux de traitement

```
Déclenchement manuel
    │
    ▼
Récupération de tous les hôtes Zabbix
    │
    ▼
Split — Traitement hôte par hôte
    │
    ▼
IF — L'hôte a-t-il uniquement le template générique par défaut ?
    │
    ├── NON → Ignoré (déjà configuré)
    │
    └── OUI
         │
         ▼
    Get System Description (Zabbix item)
         │
         ▼
    Formatage des données
         │
         ▼
    Get System Object ID (OID SNMP)
         │
         ▼
    Mapping OID → Template vendor
    (Fortinet, Cisco, Dell, Juniper, HP, Linux...)
         │
         ▼
    host.update — Assignation du bon template
    + Suppression du template générique
```

---

## Vendors supportés

| Vendor | OID SNMP |
|--------|----------|
| Fortinet (FG-60E, 80E, 100F, 200E, 400F...) | `1.3.6.1.4.1.12356` |
| Cisco | `1.3.6.1.4.1.9.1` |
| Dell iDRAC | `1.3.6.1.4.1.674.10892.5` |
| HP / Aruba | `1.3.6.1.4.1.11` |
| Juniper MX | `1.3.6.1.4.1.2636.1.1.1.2` |
| Juniper EX | `1.3.6.1.4.1.2636.1.1.1.3` |
| Extreme Networks VSP | `1.3.6.1.4.1.1916.2.14` |
| Extreme FabricEngine | `1.3.6.1.4.1.1916.2.306` |
| ADVA Optical | `1.3.6.1.4.1.2544.1.11.1.1` |
| Ubiquiti AP | `1.3.6.1.4.1.41112` |
| Linux (Net-SNMP) | `1.3.6.1.4.1.8072.3.2.10` |
| Windows Server | `1.3.6.1.4.1.311.1.1.3` |

> La table de mapping est extensible — ajoutez vos OID directement dans le nœud Code.

---

## Prérequis

- n8n >= 1.0
- Zabbix >= 7.0 avec API REST activée
- Token API Zabbix (Administration → API tokens)
- Les hôtes SNMP doivent avoir l'item **"System object ID"** actif

---

## Installation

### 1. Importer le workflow

Dans n8n : **Settings → Import workflow** → sélectionner `workflow.json`

### 2. Remplacer les placeholders

| Placeholder | Valeur |
|------------|--------|
| `YOUR_ZABBIX_HOST` | IP ou FQDN de votre serveur Zabbix |
| `YOUR_ZABBIX_API_TOKEN` | Token API Zabbix |
| `YOUR_DEFAULT_TEMPLATE_ID` | ID du template SNMP générique par défaut |
| `YOUR_TEMPLATE_ID_FORTINET` | ID du template Fortinet dans votre Zabbix |
| `YOUR_TEMPLATE_ID_CISCO` | ID du template Cisco dans votre Zabbix |
| `YOUR_TEMPLATE_ID_DELL` | ID du template Dell iDRAC dans votre Zabbix |
| *(etc.)* | *(un placeholder par vendor)* |

### 3. Trouver les IDs de templates

Dans Zabbix : **Configuration → Templates** → cliquer sur le template → l'ID est dans l'URL (`templateid=XXXXX`).

### 4. Ajouter vos propres OID

Dans le nœud **"Décider du Template Final Bis"**, ajoutez vos mappings dans le tableau `oidMappings` :

```javascript
{
  pattern: /1\.3\.6\.1\.4\.1\.VOTRE_OID/,
  templateId: 'VOTRE_TEMPLATE_ID',
  vendor: 'Nom du vendor'
}
```

---

## Notes

- Le workflow ne touche que les hôtes ayant **uniquement** le template générique par défaut (condition IF)
- Les hôtes déjà configurés avec un template vendor sont ignorés — pas de risque d'écrasement
- Le format OID `SNMPv2-SMI::enterprises.X` est automatiquement converti en `1.3.6.1.4.1.X`
- Testé sur Zabbix 7.4

---

## Licence

MIT — Libre d'utilisation, modification et distribution.

---

*Partagé par [Xavier Maugendre](https://www.linkedin.com/in/xmaugendre/) — Expert Observabilité & SRE*
