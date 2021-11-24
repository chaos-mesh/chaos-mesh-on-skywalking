# chaos-mesh-on-skywalking

## How to run a demo?

Tips: You can change configurations as needed, especially the Spring demo and exporter. Because we can not use go template to slice `service` and `serviceInstance`,  these two fileds's length depends on your namespace's name and  pod's name.

### Step 1: Install SkyWalking

```
export SKYWALKING_RELEASE_NAME=skywalking  # change the release name according to your scenario
export SKYWALKING_RELEASE_NAMESPACE=skywalking  # change the namespace to where you want to install SkyWalking
kubectl create ns skywalking
export REPO=skywalking
helm repo add ${REPO} https://apache.jfrog.io/artifactory/skywalking-helm 
helm install "${SKYWALKING_RELEASE_NAME}" ${REPO}/skywalking -n "${SKYWALKING_RELEASE_NAMESPACE}" \
  --set oap.image.tag=8.8.1 \
  --set oap.storageType=elasticsearch \
  --set ui.image.tag=8.8.1 \
  --set elasticsearch.imageTag=6.8.6 \
  --set elasticsearch.minimumMasterNodes=1 \
  --set elasticsearch.replicas=1

```

### Step 2: Deploy Spring Boot Demo and Exporter

```
git clone https://github.com/FingerLeader/chaos-mesh-on-skywalking.git
cd chaos-mesh-on-skywalking
kubectl apply -f demo-deployment.yaml
kubectl apply -f exporter-deployment.yaml
```

 ### Step 3: Install Chaos Mesh

```
helm repo add chaos-mesh https://charts.chaos-mesh.org
kubectl create ns chaos-testing
helm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-testing 
```

### Step 4: Run the Demo on the Front End

```
kubectl port-forward svc/skywalking-ui 8080:80 -n skywalking
kubectl port-forward svc/spring-boot-skywalking-demo 8079:8080 -n skywalking
kubectl port-forward svc/chaos-dashboard 2333:2333 -n chaos-testing
```

Try to access http://localhost:8079, then you can get information about your service on http://localhost:8080.

