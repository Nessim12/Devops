# Guide d'Installation - Prometheus & Grafana

Ce guide vous explique étape par étape comment intégrer Prometheus et Grafana dans votre projet Kubernetes.

## 📋 Prérequis

- Kubernetes/Minikube fonctionnel
- Namespace `devops` créé
- Accès à kubectl
- L'application Spring Boot déployée

## 🚀 Étapes d'Installation

### Étape 1 : Ajouter Spring Actuator (✅ DÉJÀ FAIT)

Spring Actuator a été ajouté au `pom.xml` et configuré dans `application.properties`.

**Vérification:**
```bash
# Rebuild et redéployer l'application
mvn clean package
# Puis redéployer dans Kubernetes
```

### Étape 2 : Déployer Prometheus

```bash
# 1. Créer la ConfigMap pour Prometheus
kubectl apply -f prometheus-configmap.yaml

# 2. Déployer Prometheus
kubectl apply -f prometheus-deployment.yaml

# 3. Vérifier le déploiement
kubectl get pods -n devops | grep prometheus
kubectl get svc -n devops | grep prometheus
```

**Attendre que le pod soit en état Running:**
```bash
kubectl get pods -n devops -w
```

### Étape 3 : Déployer Grafana

```bash
# 1. Déployer Grafana
kubectl apply -f grafana-deployment.yaml

# 2. Vérifier le déploiement
kubectl get pods -n devops | grep grafana
kubectl get svc -n devops | grep grafana
```

### Étape 4 : Accéder aux Interfaces

#### Prometheus
```bash
# Créer un tunnel
minikube service prometheus-service -n devops

# Ou utiliser le port direct
kubectl port-forward -n devops svc/prometheus-service 9090:9090
```

Accédez à: `http://127.0.0.1:9090` (ou le port affiché par minikube)

#### Grafana
```bash
# Créer un tunnel
minikube service grafana-service -n devops

# Ou utiliser le port direct
kubectl port-forward -n devops svc/grafana-service 3000:3000
```

Accédez à: `http://127.0.0.1:3000`
- **Username:** `admin`
- **Password:** `admin`

### Étape 5 : Configurer Grafana

1. **Connectez-vous à Grafana** (admin/admin)
2. **Ajouter la source de données Prometheus:**
   - Aller dans **Configuration** → **Data Sources**
   - Cliquer sur **Add data source**
   - Sélectionner **Prometheus**
   - URL: `http://prometheus-service:9090` (depuis le cluster)
   - Cliquer sur **Save & Test**

3. **Créer un Dashboard:**
   - Aller dans **Dashboards** → **New Dashboard**
   - Ajouter des panneaux (panels) pour visualiser les métriques

### Étape 6 : Configurer Jenkins (Optionnel)

Si Jenkins est accessible depuis Kubernetes:

1. **Installer le plugin Prometheus dans Jenkins:**
   - Jenkins → **Manage Jenkins** → **Plugins**
   - Rechercher "Prometheus metrics"
   - Installer et redémarrer Jenkins

2. **Vérifier l'endpoint métriques:**
   - Accédez à: `http://<jenkins-url>/prometheus`

3. **Mettre à jour la ConfigMap Prometheus:**
   ```bash
   # Modifier prometheus-configmap.yaml avec l'IP correcte de Jenkins
   # Puis recharger:
   kubectl apply -f prometheus-configmap.yaml
   kubectl delete pod -n devops -l app=prometheus  # Redémarrer Prometheus
   ```

### Étape 7 : Vérifier les Métriques

#### Vérifier Spring Boot Actuator
```bash
# Depuis le cluster
kubectl port-forward -n devops svc/springboot-service 8080:8080

# Tester l'endpoint
curl http://localhost:8080/student/actuator/prometheus
```

#### Vérifier Prometheus Targets
- Accéder à Prometheus UI
- Aller dans **Status** → **Targets**
- Vérifier que tous les targets sont **UP**

### Étape 8 : Utiliser le Dashboard HTML

1. **Ouvrir `dashboard.html`** dans votre navigateur
2. **Mettre à jour les URLs** si nécessaire dans le fichier
3. **Profiter de l'interface de monitoring!**

## 📊 Métriques Disponibles

### Spring Boot Actuator
- **JVM:** Mémoire, threads, GC
- **HTTP:** Requêtes, latence, codes de statut
- **Database:** Connexions, pool de connexions
- **Application:** Métriques personnalisées

### Jenkins
- Builds: Nombre, durée, statut
- Jobs: Succès/Échecs
- Nodes: Utilisation des agents

### Node Exporter (si installé)
- CPU, mémoire, disque
- Réseau, système de fichiers

## 🔧 Dépannage

### Prometheus ne scrape pas les métriques
```bash
# Vérifier les logs
kubectl logs -n devops -l app=prometheus

# Vérifier la configuration
kubectl get configmap prometheus-config -n devops -o yaml
```

### Grafana ne peut pas se connecter à Prometheus
- Vérifier que Prometheus est accessible: `http://prometheus-service:9090`
- Vérifier les logs Grafana: `kubectl logs -n devops -l app=grafana`

### Spring Actuator ne répond pas
```bash
# Vérifier les logs de l'application
kubectl logs -n devops -l app=springboot

# Vérifier que Actuator est activé
curl http://localhost:8080/student/actuator/health
```

## 📝 Commandes Utiles

```bash
# Voir tous les services
kubectl get svc -n devops

# Voir tous les pods
kubectl get pods -n devops

# Logs Prometheus
kubectl logs -n devops -l app=prometheus -f

# Logs Grafana
kubectl logs -n devops -l app=grafana -f

# Supprimer tout
kubectl delete -f prometheus-deployment.yaml
kubectl delete -f grafana-deployment.yaml
kubectl delete -f prometheus-configmap.yaml
```

## 🎯 Prochaines Étapes

1. ✅ Créer des dashboards Grafana personnalisés
2. ✅ Configurer des alertes dans Prometheus
3. ✅ Ajouter Node Exporter pour les métriques système
4. ✅ Intégrer avec des outils de notification (Slack, Email)

## 📚 Ressources

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html)

