# K8s namespace to deploy Kuma
NAMESPACE ?= kuma-system
CHART_REPO_NAME ?= kuma
#CHART_NAME ?= kuma/kuma
CHART_NAME ?= ./kuma
CHART_VERSION ?= 2.7.1
RELEASE ?= kuma
PROJECT ?= poc-kuma

# Default values for values and patch files
VALUES_FILE ?= $(PROJECT)-values.yaml
CONTAINER_PATCH_FILE ?= container-patch-poc-kuma.yaml

# Check for required binaries
EXECUTABLES = helm kubectl
K := $(foreach exec,$(EXECUTABLES),\
	$(if $(shell command -v $(exec) 2> /dev/null),some string,$(error "No $(exec) in PATH")))

.PHONY: init
init:
	helm plugin install https://github.com/databus23/helm-diff --version ${HELM_DIFF_VERSION} || true
	helm repo add ${CHART_REPO_NAME} https://kumahq.github.io/charts
	helm repo update

.PHONY: values
values:
	helm show values -n ${NAMESPACE} ${CHART_NAME} --version ${CHART_VERSION}

.PHONY: template
template:
	helm template -n ${NAMESPACE} ${RELEASE} ${CHART_NAME} -f ${VALUES_FILE} --version ${CHART_VERSION}

.PHONY: validate
validate:
	@helm template -n ${NAMESPACE} ${RELEASE} ${CHART_NAME} -f ${VALUES_FILE} --version ${CHART_VERSION} --validate

.PHONY: diff
diff:
	helm diff upgrade -n ${NAMESPACE} --allow-unreleased ${RELEASE} ${CHART_NAME} -f ${VALUES_FILE} --version ${CHART_VERSION} --disable-validation
	kubectl diff -n ${NAMESPACE} -f ${CONTAINER_PATCH_FILE} || true

.PHONY: deploy
deploy:
	@echo "Iniciando despliegue de Kuma..."
	# Create namespace if it doesn't exist
	kubectl get namespace ${NAMESPACE} || kubectl create namespace ${NAMESPACE}
	# Installs Gateway API CRDS
	kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml
	# Ensure CRDs are ready
	kubectl wait --for=condition=Established --timeout=60s crds --all
	# Check if the release exists and is in a failed state, delete it if necessary
	helm list -n ${NAMESPACE} | grep -q "^${RELEASE}" \
	    && helm status -n ${NAMESPACE} ${RELEASE} | grep -q "failed" \
	    && helm uninstall -n ${NAMESPACE} ${RELEASE} || true
	# Check if the release exists, and install or upgrade accordingly
	@set -e; \
	(helm list -n ${NAMESPACE} | grep -q "^${RELEASE}" \
	    && helm upgrade -n ${NAMESPACE} ${RELEASE} ${CHART_NAME} --version ${CHART_VERSION} -f ${VALUES_FILE} --timeout 10m0s \
	    || helm install -n ${NAMESPACE} ${RELEASE} ${CHART_NAME} --version ${CHART_VERSION} -f ${VALUES_FILE} --timeout 10m0s)
	# Wait for the control plane to be ready
	kubectl wait --for=condition=Ready pod -l app.kubernetes.io/name=kuma-control-plane -n ${NAMESPACE} --timeout=300s
	# Apply container patch file
	kubectl apply -n ${NAMESPACE} -f ${CONTAINER_PATCH_FILE} \
	|| { \
	    echo "Despliegue fallido. Limpiando recursos..."; \
	    make destroy; \
	    echo "Limpieza completada. Por favor, revisa los logs y corrige los errores antes de intentar nuevamente."; \
	    exit 1; \
	}

.PHONY: destroy
destroy:
	@echo "Eliminando recursos de Kuma..."
	# Delete container patch file and uninstall release
	kubectl delete -n ${NAMESPACE} -f ${CONTAINER_PATCH_FILE} --ignore-not-found=true
	helm uninstall -n ${NAMESPACE} ${RELEASE} || true
	@echo "Recursos eliminados correctamente."

.PHONY: status
status:
	# Show the status of the Helm release
	helm status -n ${NAMESPACE} ${RELEASE}
