# K8s namespace to deploy Kuma
NAMESPACE ?= kuma-system
CHART_REPO_NAME ?= kuma
CHART_NAME ?= kuma/kuma
CHART_VERSION ?= 2.7.1
RELEASE ?= kuma
PROJECT ?= nexus

ifndef ENVIRONMENT
$(error ENVIRONMENT variable must be defined.)
endif

ifndef HELM_DIFF_VERSION
$(error HELM_DIFF_VERSION variable must be defined.)
endif

VALUES_FILE ?= $(PROJECT)-$(ENVIRONMENT)-values.yaml
CONTAINER_PATCH_FILE ?= container-patch-$(ENVIRONMENT).yaml

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
		helm diff upgrade -n ${NAMESPACE} --allow-unreleased ${RELEASE} ${CHART_NAME}  -f ${VALUES_FILE} --version ${CHART_VERSION} --disable-validation
		kubectl diff -n ${NAMESPACE} -f ${CONTAINER_PATCH_FILE} || true

.PHONY: deploy
deploy:
		# Installs Gateway API CRDS
		kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml
		helm upgrade --install -n ${NAMESPACE} ${RELEASE} ${CHART_NAME} --version ${CHART_VERSION} -f ${VALUES_FILE} 
		kubectl apply -n ${NAMESPACE} -f ${CONTAINER_PATCH_FILE}


.PHONY: destroy
destroy:
		kubectl delete -n ${NAMESPACE} -f ${CONTAINER_PATCH_FILE} --ignore-not-found=true
		helm uninstall -n ${NAMESPACE} ${RELEASE} || true

.PHONY: status
status:
		helm status -n ${NAMESPACE} ${RELEASE}
