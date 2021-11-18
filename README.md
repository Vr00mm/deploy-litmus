# Deploy and test litmus chaos stack ( litmus + monitoring + target app + hub-gen)

## Prerequis

```
git clone https://github.com/Vr00mm/deploy-litmus.git
```

```
export KUBECONFIG=${HOME}/.kube/config_minikube
```

## Litmus
```
helm repo add litmuschaos https://litmuschaos.github.io/litmus-helm/
helm repo update
helm upgrade --install chaos-center litmuschaos/litmus --namespace litmus --create-namespace -f ./values/litmus/common.yaml -f ./values/litmus/env-prod.yaml
kubectl -n litmus wait --for=condition=Ready pods --all
```

## Autoconf

```
docker run --net=host --rm \
-e "KUBE_CONTEXT=minikube" \
-e "LITMUS_URL=http://127.0.0.1:9091" \
-e "LITMUS_USERNAME=admin" \
-e "LITMUS_PASSWORD=litmus" \
-v "$(pwd)/values/litmus-as-code/configuration.json:/root/configuration.json" \
-v "${HOME}/.kube/config_minikube:/root/.kube/config" \
-v "${HOME}/.minikube:${HOME}/.minikube" \
--entrypoint /bin/sh \
-it vr00mm/litmus-as-code:v0.1.0

	kubectl -n litmus port-forward svc/chaos-center-litmus-frontend-service 9091:9091 &
	bash /entrypoint.sh /root/configuration.json
```


## deploy agent
litmusctl config set-account  --endpoint "http://$(minikube ip):$(minikube kubectl -- -n litmus get svc/chaos-center-litmus-frontend-service -ojson |jq -r '.spec.ports[0].nodePort')" \
--username "admin" \
--password "litmus" 

litmusctl create agent 

## Podtato head

```
kubectl apply -f ./src/podtato-head/manifest.yaml
kubectl -n podtato-kubectl wait --for=condition=Ready pods --all
```

## Deploy monitoring

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
kubectl -n monitoring wait --for=condition=Ready pods --all
```


# Open Litmus
```
firefox http://$(minikube ip):$(minikube kubectl -- -n litmus get svc/chaos-center-litmus-frontend-service -ojson |jq -r '.spec.ports[0].nodePort')
```

# Open grafana
```
minikube kubectl -- -n monitoring port-forward svc/kube-prometheus-stack-grafana 8080:80 &
firefox http://127.0.0.1:8080
```
#admin/prom-operator


# Open podtatohead
```
firefox http://$(minikube ip):$(minikube kubectl -- -n podtato-kubectl get svc/podtato-main -ojson |jq -r '.spec.ports[0].nodePort')
```
