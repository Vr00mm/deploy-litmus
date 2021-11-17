git clone https://github.com/Vr00mm/deploy-litmus.git


## Litmus
helm repo add litmuschaos https://vr00mm.github.io
helm repo update
helm upgrade --install chaos-center litmuschaos/litmus --version "2.0.0" --namespace litmus --create-namespace -f ./values/litmus/common.yaml -f ./values/litmus/env-prod.yaml
kubectl -n litmus wait --for=condition=Ready pods --all


## Autoconf



docker run --net=host \
-e "KUBE_CONTEXT=minikube" \
-e "LITMUS_CONFIGURATION_PATH=/root/config.json" \
-e "LITMUS_URL=http://127.0.0.1:9091" \
-e "LITMUS_USERNAME=admin" \
-e "LITMUS_PASSWORD=litmus" \
-v "$(pwd)/values/litmus-as-code/configuration.json:/root/configuration.json" \
-v "${HOME}/.kube/config:/root/.kube/config" \
--entrypoint /bin/sh \
-it vr00mm/litmus-as-code:v0.1.0

	kubectl -n litmus port-forward svc/litmusportal-frontend 9091:9091 &
	bash /entrypoint.sh /root/configuration.sh
