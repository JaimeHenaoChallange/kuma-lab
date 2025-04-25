Aquí tienes un README completo y detallado para el repositorio kuma-lab, basado en todo lo que hemos conversado y la estructura del proyecto:

---

# Kuma Lab

Este repositorio contiene los recursos necesarios para desplegar y gestionar Kuma, un service mesh universal, en un clúster de Kubernetes utilizando Helm y ArgoCD. Incluye configuraciones para el control plane de Kuma, políticas de tráfico, inyección automática de sidecars, y ejemplos de aplicaciones.

---

## Tabla de Contenidos

1. Descripción General
2. Estructura del Proyecto
3. Requisitos Previos
4. Instalación
5. Configuración de Kuma
6. Despliegue de Aplicaciones
7. Políticas de Kuma
8. Acceso al Dashboard de Kuma
9. Gestión con ArgoCD
10. Eliminación de Recursos
11. Referencias

---

## Descripción General

Kuma es un service mesh universal que permite gestionar el tráfico entre servicios de manera segura y eficiente. Este repositorio utiliza Helm para desplegar el control plane de Kuma y ArgoCD para gestionar la sincronización de recursos en Kubernetes.

---

## Estructura del Proyecto

```plaintext
kuma-lab/
├── argocd/
│   └── kuma-argocd-application.yaml  # Configuración de ArgoCD para Kuma
├── Docker/
│   └── Dockerfile                   # Dockerfile para construir imágenes personalizadas
├── helm/
│   ├── Chart.yaml                   # Helm chart principal
│   ├── container-patch-poc-kuma.yaml # Configuración de patches para contenedores
│   ├── makefile                     # Makefile para automatizar despliegues
│   ├── poc-kuma-values.yaml         # Valores personalizados para Kuma
│   ├── values.yaml                  # Valores predeterminados del chart
│   ├── kuma/
│   │   ├── Chart.yaml               # Helm chart del control plane de Kuma
│   │   ├── crds/                    # CRDs de Kuma
│   │   ├── templates/               # Plantillas de recursos de Kubernetes
│   │   └── values.yaml              # Valores predeterminados del control plane
│   └── kuma-resources-default-mesh/ # Recursos adicionales para el mesh
```

---

## Requisitos Previos

1. **Kubernetes**: Un clúster de Kubernetes (v1.21 o superior).
2. **Helm**: Instalado y configurado en tu máquina local.
3. **ArgoCD**: Instalado en el clúster para gestionar aplicaciones.
4. **Docker**: Para construir imágenes personalizadas si es necesario.
5. **kubectl**: Instalado y configurado para interactuar con el clúster.

---

## Instalación

### 1. Desplegar Kuma con Helm

Ejecuta el siguiente comando para desplegar Kuma en el namespace `kuma-system`:

```bash
make deploy
```

Este comando:
- Crea el namespace `kuma-system`.
- Instala los CRDs necesarios.
- Despliega el control plane de Kuma utilizando Helm.

### 2. Configurar ArgoCD

Aplica la configuración de ArgoCD para sincronizar Kuma:

```bash
kubectl apply -f argocd/kuma-argocd-application.yaml
```

Esto permitirá que ArgoCD gestione el despliegue de Kuma.

---

## Configuración de Kuma

### Habilitar la Inyección Automática de Sidecars

Para habilitar la inyección automática de sidecars en un namespace, ejecuta:

```bash
kubectl annotate namespace <namespace> kuma.io/sidecar-injection=enabled
```

Por ejemplo, para el namespace `poc`:

```bash
kubectl annotate namespace poc kuma.io/sidecar-injection=enabled
```

---

## Despliegue de Aplicaciones

Ejemplo de despliegue de aplicaciones en el namespace `poc`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
  namespace: poc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
        - name: app1
          image: hashicorp/http-echo
          args:
            - "-text=Hello from app1"
          ports:
            - containerPort: 5678
```

Aplica el manifiesto:

```bash
kubectl apply -f <path-to-your-app-manifest>.yaml
```

---

## Políticas de Kuma

### Permitir Tráfico Entre Servicios

Crea una política `TrafficPermission` para permitir el tráfico entre servicios:

```yaml
apiVersion: kuma.io/v1alpha1
kind: TrafficPermission
metadata:
  name: allow-all
  namespace: poc
spec:
  sources:
    - match:
        kuma.io/service: '*'
  destinations:
    - match:
        kuma.io/service: '*'
```

Aplica la política:

```bash
kubectl apply -f traffic-permission.yaml
```

### Registrar Tráfico

Crea una política `TrafficLog` para registrar el tráfico:

```yaml
apiVersion: kuma.io/v1alpha1
kind: TrafficLog
metadata:
  name: log-all
  namespace: poc
spec:
  sources:
    - match:
        kuma.io/service: '*'
  destinations:
    - match:
        kuma.io/service: '*'
```

Aplica la política:

```bash
kubectl apply -f traffic-log.yaml
```

---

## Acceso al Dashboard de Kuma

Expon el dashboard de Kuma para acceder a la interfaz gráfica:

```bash
kubectl port-forward svc/kuma-control-plane -n kuma-system 5681:5681
```

Abre tu navegador en `http://localhost:5681/gui`.

---

## Gestión con ArgoCD

1. Accede a la interfaz de ArgoCD.
2. Sincroniza la aplicación `kuma` para aplicar cambios.
3. Monitorea el estado de los recursos desde la interfaz.

---

## Eliminación de Recursos

Para eliminar Kuma y todos los recursos asociados, ejecuta:

```bash
make destroy
```

Esto eliminará los recursos creados por Helm y las configuraciones aplicadas.

---

## Referencias

- [Documentación Oficial de Kuma](https://kuma.io/docs/)
- [Helm Charts de Kuma](https://github.com/kumahq/charts)
- [ArgoCD](https://argo-cd.readthedocs.io/)

---

Con este README, tienes toda la información necesaria para trabajar con Kuma en tu clúster de Kubernetes. Si necesitas más ayuda, no dudes en preguntar. 🚀