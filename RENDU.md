# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom :** Hamza El Ferdi
**Groupe :**

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > 3 nœuds : `k3d-lab-server-0` (control-plane) et deux workers, `k3d-lab-agent-0` et `k3d-lab-agent-1`. Ils sont tous `Ready`.
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   > `docker ps` montre un conteneur pour le serveur, deux conteneurs pour les agents et un load balancer. Dans k3d, un nœud Kubernetes est concrètement un conteneur Docker qui exécute K3s.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   > Un Deployment `web`, un ReplicaSet géré par ce Deployment et un pod nginx. Le Service `web` a ensuite fourni une adresse stable.
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > Le Deployment a détecté que le nombre de réplicas était insuffisant et a créé automatiquement un nouveau pod via son ReplicaSet.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non. La modification avait été faite directement dans le système de fichiers du conteneur ; ce stockage est éphémère et le nouveau pod est recréé depuis l'image nginx.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   > Le tunnel était attaché au pod supprimé. Il faut relancer le `port-forward`, idéalement vers le Service.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > `create` crée une ressource une seule fois de manière impérative. `apply` déclare l'état souhaité dans des fichiers et peut être rejoué pour mettre le cluster à jour.
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Il sélectionne les pods auxquels le Service envoie le trafic grâce aux labels. S'il ne correspond à aucun pod, le Service n'a aucun endpoint et ne peut pas servir l'application.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   > Les trois réplicas sont répartis sur `k3d-lab-agent-0`, `k3d-lab-agent-1` et `k3d-lab-server-0`.
   >
   > ```text
   > site-74c446dbb9-5jzfd   1/1   Running   0   ...   k3d-lab-agent-1
   > site-74c446dbb9-5rgzd   1/1   Running   0   ...   k3d-lab-server-0
   > site-74c446dbb9-wjsbw   1/1   Running   0   ...   k3d-lab-agent-0
   > ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > Le nouveau pod était en `ErrImagePull` (puis `ImagePullBackOff`). `kubectl describe pod <nom-du-pod>` affichait l'échec de récupération de l'image.
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Oui, les anciens pods sont restés disponibles pendant le rolling update ; la readiness probe empêchait le pod cassé de recevoir du trafic. Après le rollback puis `kubectl apply -f k8s/site/`, le cluster et les fichiers sont revenus à `nginx:1.27`.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    > https://fantastic-space-invention-qv7gx6j6r6267rg-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > Oui. Portainer et `kubectl` utilisent tous les deux l'API Server Kubernetes et manipulent les mêmes objets, notamment le Deployment `mario`.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    > Bonus A réalisé : Ingress `site` fonctionnel sur le port 8081. Bonus B réalisé : readiness probe HTTP sur `/` et requests/limits CPU et mémoire.

---

**Score obtenu avec `./scripts/check.sh` :** 18 / 18
