# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom : Enzo Caçote**
**Groupe : RS2 **

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > kubectl get nodes affiche 3 nœuds : k3d-lab-server-0 (rôle control-plane, le master, qui héberge l’API Server et décide du placement des pods) et k3d-lab-agent-0 et k3d-lab-agent-1 (les workers, qui exécutent les pods applicatifs)
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   >docker ps montre 3 conteneurs pour les nœuds (+ le load balancer k3d-lab-serverlb et éventuellement un registry ou tools). Dans k3d, un nœud est concrètement un conteneur Docker qui exécute k3s, une distribution Kubernetes légère. Ce n’est pas une vraie machine.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   >kubectl create deployment a créé un Deployment (web), qui a créé un ReplicaSet (web-xxxxx), lequel a créé un Pod (web-xxxxx-yyyyy) Le kubectl expose a ajouté un Service (web) Le kubectl get all affiche ces objets
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   >Le pod supprimé a été immédiatement recréé, avec un nouveau nom. Le ReplicaSet doit maintenir le nombre de réplicas demandé (1) : quand l’état réel (0 pod) diffère de l’état désiré (1 pod), il corrige. C’est la boucle de réconciliation

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   >Non, le message n’a pas survécu. Le système de fichiers d’un conteneur est éphémère : le nouveau pod est créé à partir de l’image nginx d’origine, donc la page par défaut est revenue. La modification faite avec exec n’existait que dans l’ancien conteneur.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   >Un port-forward vers un Service s’attache en réalité à un seul pod. Quand ce pod est supprimé, le tunnel perd sa cible et casse (l’erreur est visible dans /tmp/web.log). Il faut le relancer vers le nouveau pod.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   >kubectl create est impératif : il crée un objet et échoue s’il existe déjà. kubectl apply -f est déclaratif : on décrit l’état désiré dans un fichier, et Kubernetes le crée ou le met à jour pour y correspondre. Il peut être rejoué sans erreur et le fichier peut être versionné dans Git.
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   >Le selector d’un Service indique à quels pods envoyer le trafic, grâce à leurs labels. S’il ne correspond à aucun pod, le Service n’a aucun endpoint : il existe, mais les requêtes n’aboutissent pas (visible avec kubectl get endpoints site, qui affiche <none>).

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   ```
   /workspaces/atelier-kubernetes (main) $ kubectl get pods -l app=site -o wide
NAME                    READY   STATUS    RESTARTS   AGE    IP          NODE               NOMINATED NODE   READINESS GATES
site-5cc664db9c-dxzwv   1/1     Running   0          103s   10.42.1.6   k3d-lab-server-0   <none>           <none>
site-5cc664db9c-jnvwv   1/1     Running   0          103s   10.42.0.6   k3d-lab-agent-1    <none>           <none>
site-5cc664db9c-mrgrt   1/1     Running   0          103s   10.42.2.5   k3d-lab-agent-0    <none>           <none>
Les 3 pods sont répartis sur les 3 nœuds du cluster : un sur `k3d-lab-server-0`, un sur `k3d-lab-agent-0` et un sur `k3d-lab-agent-1`. C'est le scheduler de Kubernetes qui choisit le placement, en répartissant les pods sur les nœuds disponibles
   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    >Le pod était en ErrImagePull puis ImagePullBackOff. La commande qui a donné la cause est kubectl describe pod -l app=site (section Events, qui indique que l’image nginx:9.99 est introuvable).
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    >Oui, le site restait accessible. Pendant un rolling update, Kubernetes ne supprime les anciens pods qu’une fois les nouveaux prêts. Le pod cassé n’est jamais devenu prêt, donc les anciens pods ont continué à servir le trafic.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    >https://cuddly-giggle-r44r7g6qrwjvcx5q6-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    >Oui, le Deployment `mario` créé depuis Portainer est bien visible avec `kubectl get deploy mario` (et `kubectl get deploy mario -o yaml` pour le détail). C'est un Deployment Kubernetes standard, comme ceux créés en ligne de commande.
   
    > Portainer et `kubectl` parlent au même composant : l'**API Server** du cluster (sur le nœud control plane, ici `k3d-lab-server-0`). C'est le point d'entrée unique de Kubernetes : `kubectl` lui envoie des requêtes depuis le terminal, et Portainer, qui tourne dans le cluster avec les droits nécessaires, lui envoie les mêmes requêtes depuis son interface web. Les deux outils ne sont que des clients de la même API, donc ils voient et modifient les mêmes objets.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    >

---

**Score obtenu avec `./scripts/check.sh` :** ... / ...
