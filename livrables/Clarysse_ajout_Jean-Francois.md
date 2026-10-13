# Clarysse · Ajout de Jean-François STOECKEL (transfert + mails)

Date : 13/10/2026. Demande de Steve : relire les retours d'appels de Clarysse, ajouter Jean-François
au transfert d'appel avec son numéro pro, et l'ajouter aux mails de synthèse qui le concernent.

## Coordonnées à utiliser

| Personne | Rôle | Numéro pro (format Retell) | E-mail |
|---|---|---|---|
| Jean-François STOECKEL | Responsable SAV et planning | `+33620625302` (06 20 62 53 02) | jf.s@koenig-automatisme.fr |

Le 03 88 22 76 81 de sa signature est le standard de l'agence, pas son numéro direct : ne pas l'utiliser
comme cible de transfert (voir ci-dessous pourquoi).

## Ce que montrent les retours d'appels (24/09 → 13/10)

Destinataires actuels de tous les mails « Clarysse [..] : appel du … » : steve.k@ et jordan.f@ uniquement.
Jean-François n'en reçoit aucun alors que la signature de Steve le désigne depuis le 05/10 pour
« Validation des devis / demande d'intervention ».

**Le transfert vers « notre technicien » ne fonctionne pas.** Trois cas sur trois :

| Appel | Ce qui s'est passé |
|---|---|
| 24/09 10h37 [URGENT], porte Normsthal à Marlenheim (06 77 80 25 46) | « Le technicien n'a pas pu décrocher tout de suite », repli sur prise de message. |
| 24/09 11h36 [Depannage], Paul FADY / CBA, chantier Poste Holsheim | La cible du transfert a répondu « société Koenig Automatisme bonjour » : la ligne transférée est le standard 03 88 22 76 81, l'appel reboucle sur l'agence (ou sur Clarysse elle-même). Clarysse a conclu « je n'ai pas réussi à joindre le technicien ». |
| 13/10 14h39 [URGENT], Alain, hôpitaux de Strasbourg Hautepierre, sas ambulance (03 88 12 73 65) | « Je vous mets en relation avec notre technicien, un instant » puis fin de l'appel après 1 min 14 s. Aucune trace que quelqu'un ait décroché, pas de rappel confirmé au client. À rappeler en priorité. |

Conclusion : la cible de transfert configurée est le standard, pas un portable. Mettre le portable de
Jean-François comme cible résout le cas 2 ; pour les cas 1 et 3 il faut aussi que le téléphone soit
joignable (sonnerie, pas de renvoi sur messagerie pendant le transfert Retell, qui attend ~20 à 30 s).

**Autres points relevés dans les transcriptions :**
- Beaucoup d'appels de 1 à 12 s sans échange (raccroché à l'annonce « nous avons beaucoup d'appels ») :
  13/10 08h34 ×2, 09h31, 14h30, 14h41 ; 12/10 17h31 ; 25/09 08h14. L'annonce d'accueil fait fuir une partie des appelants.
- Promesses de rappel « dans la journée » non suivies par un responsable identifié : Daniel POIRIER
  (devis portail attendu depuis 3 semaines, 2 interventions Solitech en septembre), Serge MEULEWATER / AU VIOLON
  (pièce Novoferm commande SKNWBBUJM du 04/10), Doris GRASSL (portail SEA bloqué, Reichstett, RDV SAV).
  Ce sont exactement les dossiers SAV/planning de Jean-François : d'où l'intérêt de le mettre en destinataire.
- Clarysse a classé « Monsieur Walker » pour Serge Meulewater (le fichier PrestaShop donne le bon nom) : l'identification
  par numéro fonctionne, le résumé reprend le nom prononcé. Mineur.
- Appel Engie bureau d'étude (13/10 10h37) : demande de « réactif » pour le gérant, hors sujet Koenig, probablement
  démarchage ; rien à faire.

## Modification à appliquer

Configuration sur le VPS OVH, appli Laravel « SaaS Commande » (`/var/www/saas-commande`, commande.koenig-automatisme.fr),
qui pilote l'agent Retell et envoie les mails depuis clarysse.ia@koenig-automatisme.fr.

### 1. Transfert d'appel
- Cible du transfert « technicien » (fonction transfer_call de l'agent Retell, ou variable d'environnement /
  table de paramètres côté Laravel) : remplacer le numéro actuel (standard) par `+33620625302`.
- Si l'agent gère plusieurs cibles (urgence, dépannage, commercial) : mettre Jean-François sur **URGENT** et
  **Depannage** ; laisser Steve (`+33698406377`) en secours si une cascade est possible.
- Vérifier le format E.164 (`+336…`), pas « 06 … ».

### 2. Destinataires des mails de synthèse d'appel
- Ajouter `jf.s@koenig-automatisme.fr` aux destinataires des mails « Clarysse [Depannage] », « [URGENT] »,
  « [Appel] » (messages et demandes de rappel) et « [Commande] » (suivi de commandes clients) : tout ce qui touche
  le SAV et le planning.
- Ne pas l'ajouter aux alertes du module Sous-traitants (« Chantier dans N jours », « a déposé N documents ») :
  il les reçoit déjà par un autre canal (il a transféré celle du 12/10 à Steve).
- Question ouverte pour Steve : garder jordan.f@ dans la liste ? Les mails lui sont encore adressés alors que son poste
  a été repris.

### 3. Où chercher dans l'appli (commandes à lancer depuis la session locale de Steve, `ssh ovh`)
```
ssh ovh 'grep -rn "jordan.f@koenig-automatisme.fr" /var/www/saas-commande --include=*.php --include=.env* --include=*.json --include=*.yaml -l'
ssh ovh 'grep -rn "+33388227681\|0388227681\|transfer" /var/www/saas-commande/app /var/www/saas-commande/config /var/www/saas-commande/.env -l'
```
Puis éditer le fichier trouvé (ou la table de paramètres), vider le cache de config (`php artisan config:clear`)
et, si le numéro de transfert est stocké côté Retell, mettre à jour l'outil transfer_call de l'agent dans le
tableau de bord Retell.

### 4. Test après modification
1. Appeler le standard depuis un portable, demander une « panne urgente sur une porte », vérifier que le
   06 20 62 53 02 sonne et que la mise en relation aboutit.
2. Vérifier que le mail « Clarysse [URGENT] » de ce test arrive bien à Jean-François.
3. Rappeler Alain (hôpitaux de Strasbourg Hautepierre, 03 88 12 73 65, sas ambulance) : son appel du 13/10 14h39
   n'a vraisemblablement pas été pris.

## Pourquoi ce n'est pas encore appliqué

Cette session cloud n'a pas pu ouvrir la connexion SSH vers le VPS OVH (commande refusée par le mode
automatique : accès à des identifiants et lecture de production). La modification doit être faite depuis la
session Claude Code locale de Steve (qui a `ssh ovh`), ou en autorisant la commande `ssh ovh …` dans cette session.
