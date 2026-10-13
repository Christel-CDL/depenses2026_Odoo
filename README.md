# Justificatifs de frais 2026 → Odoo (boîte de réception Achats)

Ce dépôt documente comment les justificatifs de frais de **CDL EXPERT** et de
**CDL EXPERTISES** passent d'Airtable vers Odoo, où se font les rapprochements
bancaires. Il ne contient aucune pièce comptable ni aucune donnée personnelle.

## Architecture (état au 13/10/2026)

| Élément | Rôle | Destinataires |
|---|---|---|
| Base Airtable `CDL EXPERT-COMPTA2026`, table `Justificatifs` | Registre des justificatifs (179 fiches entre le 24/02 et le 04/10/2026) | — |
| Scénario Make « Enregistrement justificatifs comptable » (quotidien, 08:00) | Copie sur Google Drive, envoi Gmail, puis `Transfert = OUI` | Welyb uniquement |
| Automatisation Airtable « Automation 1 » (à la création d'une fiche) | Envoi Outlook | Welyb **et** Odoo |
| **Nouvelle** automatisation Airtable « Reprise historique → Odoo (justificatifs) » | Reprise de l'historique, envoi Outlook | Odoo uniquement |

Boîtes Odoo (alias de journal d'achats) :

- CDL EXPERTISES → `purchases@cdl-expertises.odoo.com` (société n° 1, journal Achats n° 9)
- CDL EXPERT → `purchases-cdl-expert@cdl-expertises.odoo.com` (société n° 2, journal Achats n° 16)
- PERSO → aucun envoi

Base Odoo Online : `https://cdl-expertises.odoo.com`. Les anciennes adresses en
`@cdl-expert.odoo.com` étaient rejetées par Odoo (`550 5.1.2 Unknown host`), car
aucune base ne porte ce nom. Le 13/10/2026, le domaine d'alias
`cdl-expertises.odoo.com` a été créé dans Odoo et attribué **uniquement** aux deux
alias d'achats ; ceux-ci acceptent les courriels de « Tout le monde ». Les autres
alias, ainsi que les adresses catchall, de rebond et d'expédition, n'ont pas été
modifiés.

## Pourquoi Airtable plutôt que Make ou n8n

1. **Les données et les pièces sont déjà dans Airtable.** L'action Outlook joint
   directement le champ `Fichier`, sans téléchargement HTTP intermédiaire.
2. **Les fiches créées depuis peu partent déjà vers Odoo** grâce à
   « Automation 1 ». Il ne reste à traiter que l'historique : c'est une opération
   ponctuelle, il n'y a pas de nouveau flux permanent à maintenir.
3. **Make** consommerait des crédits pour chaque fiche (téléchargement, puis
   envoi, puis mise à jour). En outre, le scénario actuel joint les fichiers sans
   extension (`filename = ID_Justificatif`), ce qui peut empêcher Odoo de
   reconnaître le type de la pièce.
4. **n8n sur le VPS** ajoute un serveur à sécuriser et à mettre à jour, et
   transfère des pièces comptables vers un hébergement de plus. Pour un envoi
   ponctuel de 179 courriels, ce coût n'est pas justifié. n8n devient
   intéressant pour un traitement avancé, par exemple l'API Odoo, le rattachement
   automatique aux lignes bancaires ou l'OCR.

## Procédure de reprise de l'historique

L'automatisation se déclenche quand **« Envoyer à Odoo » est coché**, que
**« Envoyé Odoo le » est vide** et que **« Fichier » n'est pas vide**. Elle :

1. choisit la boîte Odoo d'après le champ `Structure` ;
2. envoie le courriel par Outlook, avec la pièce jointe d'origine ;
3. horodate `Envoyé Odoo le`. Une fiche déjà horodatée n'est jamais renvoyée.

Étapes :

1. Dans Odoo, vérifier pour chacun des deux journaux d'achats que l'alias
   accepte les courriels de `christel.lacome@cdl-expertises.com` (paramètre
   « Accepter les e-mails de »).
2. Activer l'automatisation dans Airtable (Automatisations → *Reprise
   historique → Odoo* → **On**).
3. **Test** : cocher « Envoyer à Odoo » sur une seule fiche de chaque structure,
   puis contrôler dans Odoo que la facture fournisseur brouillon est créée et que
   la pièce est lisible.
4. **Reprise** : cocher la case sur les fiches restantes, par lots de 25 à 50 pour
   rester sous les limites d'envoi d'Outlook, en excluant celles qui figurent déjà
   dans Odoo.
5. Contrôler ensuite que le nombre de brouillons dans Odoo correspond au nombre de
   fiches horodatées.
6. Désactiver l'automatisation une fois la reprise terminée.

## Points de vigilance

- **Envois en double à Welyb** : « Automation 1 » (Outlook, à la création) et le
  scénario Make (Gmail, chaque jour) envoient tous deux chaque justificatif à
  Welyb. Il faut en conserver un seul.
- « Automation 1 » : adresses Odoo corrigées dans le brouillon (bon domaine, sans
  l'espace initiale). Il faut publier la mise à jour dans l'interface Airtable pour
  qu'elle s'applique.
- Odoo, courriels sortants : le serveur Microsoft configuré échoue (jeton
  d'accès expiré). Cela n'empêche pas la réception des justificatifs, mais
  bloque les courriels envoyés par Odoo.
- Le corps de courriel du scénario Make contient des variables mal mappées
  (`{{28.Date}}{{du}}{{justificatif}}`, `{{28.Montant}}{{(TTC)}}`).
- RGPD : les pièces ne transitent que par des services déjà utilisés
  (Airtable, Microsoft 365, Odoo). Aucune copie n'est faite vers un service
  supplémentaire.
