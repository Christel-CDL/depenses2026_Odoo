# Prompt ChatGPT : contrôle des brouillons de factures fournisseurs (CDL EXPERTISES)

Copier tout le texte ci-dessous dans ChatGPT, connecté à Odoo.

---

## Contexte

Base Odoo Online : https://cdl-expertises.odoo.com.

Les justificatifs de dépenses 2026 de la société **CDL EXPERTISES** (société n° 1) arrivent par
e-mail sur l'alias `purchases@cdl-expertises.odoo.com`, qui alimente le journal **Achats n° 9**.
Odoo crée pour chacun une **facture fournisseur en brouillon** (`move_type = in_invoice`) et lit la
pièce jointe par OCR (numérisation).

Environ 107 justificatifs sont en cours d'envoi depuis le 13/10/2026 ; il s'agit d'une reprise de
l'historique de février à octobre 2026. Chaque e-mail a pour objet :
`Justificatif AAAA-MM-JJ_FOURNISSEUR_MONTANTE.ext`.
Exemple : `Justificatif 2026-04-20_HELLOASSO-SOLTENA_35.5E.pdf`, c'est-à-dire la date de la pièce,
le nom du fournisseur et le montant TTC.
Cet objet est la référence de contrôle saisie au moment de l'enregistrement de la dépense.

Problèmes déjà constatés sur les premiers brouillons :
- le **fournisseur** est souvent faux, car Odoo reprend l'expéditeur de l'e-mail
  (`christel.lacome@cdl-expertises.com`, contact « CDL EXPERTISES ») ou un texte parasite
  comme « @De : christel.lacome@cdl-expertises.com » ;
- certaines photos de tickets ne sont **pas lues** : montant à 0,00 €, date du jour ;
- les montants enregistrés dans le nom de fichier peuvent eux-mêmes être faux. Exemple :
  attestation HelloAsso n° 181107438, où le nom indiquait 65,5 € alors que la pièce porte
  **35,50 €, dont 0,50 € de contribution volontaire à HelloAsso**.
  **La pièce justificative fait toujours foi.**

## Mission

### Étape 1 – Inventaire, en lecture seule. Ne modifie rien.

Liste toutes les factures fournisseurs **en brouillon** de la société CDL EXPERTISES, journal
Achats n° 9, créées depuis le 13/10/2026. Pour chacune, présente un tableau avec :

| Colonne | Contenu |
|---|---|
| ID Odoo | identifiant du brouillon |
| Objet de l'e-mail | référence « Justificatif … » (dans le fil de discussion de la facture) |
| Date attendue / fournisseur attendu / montant attendu | extraits de l'objet |
| Date lue / fournisseur actuel / montant TTC lu / TVA lue | valeurs actuelles du brouillon |
| État OCR | lu, en cours, non lu ou erreur |
| Pièce jointe | présente ou absente |
| Écart | date, montant, fournisseur ou TVA |
| Doublon possible | même fournisseur + même date + même montant, ou même pièce reçue deux fois |

Si une facture n'a pas été lue, indique-le sans inventer de valeur.

### Étape 2 – Analyse des écarts

Pour chaque écart, indique ce que montre **la pièce elle-même** (texte OCR ou contenu du PDF) et
propose la valeur à retenir. Classe les cas :

- **A – Fournisseur seul à corriger.** La date et le montant lus concordent avec l'objet.
- **B – Écart de montant ou de date.** Indique la valeur lue sur la pièce. Ne tranche pas
  toi-même : je déciderai.
- **C – Pièce non lue (montant 0,00 €).** Propose de relancer la numérisation.
- **D – Doublon probable.** Ne supprime rien.

### Étape 3 – Mise à jour du fournisseur. Uniquement après mon accord explicite sur la liste.

Pour les cas A (et les cas B ou C une fois que je les aurai validés) :

1. Recherche le fournisseur existant dans les contacts (`res.partner`) d'après le nom figurant sur
   la pièce, comme ASF, TotalEnergies, SNCF ou une enseigne de restaurant.
   Règles de rapprochement :
   - l'organisateur réel pour les paiements via HelloAsso. Par exemple « HELLOASSO-SOLTENA »
     correspond au fournisseur **SOLTENA** ;
   - l'enseigne ou la raison sociale imprimée sur le ticket ;
   - **jamais** CDL EXPERTISES, CDL EXPERT, Christel LACOME ou une adresse e-mail.
2. S'il existe plusieurs fournisseurs proches (doublons de contacts), signale-le et prends le plus
   complet, celui qui porte un SIREN ou un n° de TVA.
3. S'il n'existe aucun fournisseur correspondant, **propose** sa création : nom, et SIREN, TVA ou
   adresse s'ils figurent sur la pièce. Attends mon accord avant de le créer.
4. Mets à jour uniquement le champ **Fournisseur** (`partner_id`) du brouillon. Contrôle ensuite
   qu'Odoo n'a pas modifié automatiquement le compte de charge, les taxes ou les conditions de
   paiement à cause du changement de fournisseur. Si c'est le cas, signale-le.

### Étape 4 – Cas particuliers à signaler (sans modifier)

- **Attestations HelloAsso.** La contribution volontaire versée à HelloAsso (par exemple 0,50 €)
  est un montant distinct de la prestation. Propose de la mettre sur une ligne séparée, sans TVA,
  et laisse-moi choisir le compte.
- **Tickets sans TVA apparente, ou TVA lue incohérente avec le TTC.** Signale-les.
- **Pièces illisibles ou incomplètes.** Signale-les.

### Étape 5 – Compte rendu

Termine par :
- le nombre de brouillons contrôlés, de fournisseurs corrigés, de fournisseurs créés, d'écarts de
  montant ou de date, de pièces non lues et de doublons probables ;
- la liste des actions qui attendent ma décision.

## Règles impératives

- **Ne valide, ne comptabilise, ne supprime et n'annule aucune facture ni aucune écriture.** Toutes
  les factures restent en brouillon.
- Ne modifie que le champ Fournisseur, et seulement après mon accord. Les autres champs (montant,
  date, TVA, lignes) ne se modifient que sur ma demande explicite, au cas par cas.
- Ne touche pas aux factures de CDL EXPERT (société n° 2, journal n° 16) ni aux autres journaux.
- N'effectue aucun rapprochement bancaire.
- Si une information n'est pas accessible avec tes droits, dis-le au lieu de supposer.
- Traite les factures par lots de 20 au maximum et fais une pause entre chaque lot pour que je
  valide.
