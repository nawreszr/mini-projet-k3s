# Mini-projet 1 : Application Web à Deux Niveaux sur K3s

## 📖 Description du Projet
Ce projet a été réalisé dans le cadre du cours d'orchestration de conteneurs avec Kubernetes (K3s). L'objectif est de déployer une application web composée de deux briques (un frontend et une API) de manière déclarative (via des manifestes YAML), de les faire communiquer en interne, et de les exposer à l'extérieur via un Ingress Controller (Traefik).

## 🏗️ Architecture

L'architecture repose sur un point d'entrée unique (Traefik) qui route le trafic vers les Services internes (ClusterIP), qui eux-mêmes distribuent la charge vers les Pods.

```text
                 Utilisateur (Windows)
                         |
                  Ingress (Traefik)
              web.k3s.local | api.k3s.local
                 /                 \
     Service frontend-svc     Service api-svc
          (ClusterIP)           (ClusterIP)
               |                     |
     Deployment frontend     Deployment api
       (2 réplicas)           (3 réplicas)
 🛠️ Prérequis
Un cluster Kubernetes (K3s) fonctionnel avec au moins 1 nœud master et 1 nœud worker.

L'Ingress Controller Traefik installé (fourni par défaut avec K3s).

kubectl configuré sur la machine d'administration.

Un accès administrateur au fichier hosts de la machine cliente (Windows) pour résoudre les noms de domaine locaux.
📂 Structure du Dépôt
text
mini-projet/
├── README.md
└── manifests/
    ├── api-deployment.yaml       # Déploiement de l'API (Podinfo)
    ├── frontend-deployment.yaml  # Déploiement du Frontend (Nginx)
    ├── services.yaml             # Définition des Services ClusterIP
    └── ingress.yaml              # Règles de routage Traefik
🚀 Guide de Déploiement
1. Création du Namespace
Toutes les ressources sont isolées dans un namespace dédié au projet.

bash
kubectl create namespace projet-devops
kubectl config set-context --current --namespace=projet-devops
2. Application des Manifestes
Déploiement de l'ensemble des ressources (Deployments, Services, Ingress) :

bash
kubectl apply -f manifests/
3. Vérification du Déploiement
Vérifier que les 5 pods (3 API + 2 Frontend) sont en statut Running :

bash
kubectl get pods -o wide
kubectl get svc
kubectl get ingress
4. Configuration Locale (Fichier Hosts)
Pour accéder aux applications depuis un navigateur, ajoutez les entrées suivantes dans le fichier C:\Windows\System32\drivers\etc\hosts de votre machine Windows (en tant qu'administrateur) :

text
192.168.78.131  web.k3s.local api.k3s.local
(Remplacez 192.168.78.131 par l'IP d'un de vos nœuds K3s).

🧪 Tests et Vérifications
Test des applications externes
Ouvrez un navigateur ou un terminal PowerShell :

powershell
# Doit afficher la page d'accueil Nginx
curl.exe http://web.k3s.local

# Doit afficher le JSON de l'API Podinfo
curl.exe http://api.k3s.local
Test de communication interne (DNS)
Vérification que le frontend peut joindre l'API par son nom DNS interne :

bash
# Récupérer le nom d'un pod frontend
kubectl get pods -l app=frontend

# Exécuter une requête depuis le pod frontend
kubectl exec -it <nom-du-pod-frontend> -- curl http://api-svc
🔄 Mise à jour sans coupure (Rolling Update)
Le projet implémente une stratégie de mise à jour continue. Pour mettre à jour l'API vers une nouvelle version :

bash
kubectl set image deployment/api podinfo=ghcr.io/stefanprodan/podinfo:6.6.0
kubectl rollout status deployment/api
Rollback (Retour arrière)
En cas de problème avec la nouvelle version, il est possible de revenir à l'état précédent :

bash
kubectl rollout undo deployment/api
🩺 Diagnostic et Résolution de panne
Scénario testé : Modification erronée du sélecteur du Service API.

Symptôme : Erreur 503 Service Unavailable sur l'Ingress et Endpoints vides.

Commande de diagnostic : kubectl describe service api-svc (a révélé un sélecteur app=api-typo).

Résolution : Correction du sélecteur via kubectl patch service api-svc -p '{"spec":{"selector":{"app":"api"}}}'.

🧹 Nettoyage
Pour supprimer toutes les ressources du projet :

bash
kubectl delete namespace projet-devops
kubectl config set-context --current --namespace=default
(N'oubliez pas de supprimer les lignes ajoutées dans votre fichier hosts Windows).
