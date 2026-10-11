Prometheus i Grafana — Kubernetes / Tanzu
1. Opis projektu

Repozytorium zawiera pliki konfiguracyjne służące do wdrożenia monitoringu w klastrze Tanzu-Kubernetes z wykorzystaniem Prometheusa i Grafany.

Prometheus jest instalowany jako pakiet VMware Tanzu, zarządzany przez mechanizm pakietów Carvel. Grafana jest przygotowana do wdrożenia za pomocą Helm.

2. Struktura repozytorium
.
├── HELM/
│   └── Grafana/
│       ├── Chart.lock
│       ├── Chart.yaml
│       ├── templates/
│       │   └── tls-secret.yaml
│       └── values.yaml
├── packages/
│   └── prometheus/
│       ├── clusterrolebinding.yaml
│       ├── clusterrole.yaml
│       ├── ingress.yaml
│       ├── kustomization.yaml
│       ├── packageinstall.yaml
│       ├── serviceaccount.yaml
│       ├── tls-secret.yaml
│       └── values-secret.yaml
└── README.md

Katalog HELM/Grafana

Zawiera konfigurację wdrożenia Grafany za pomocą Helm.

Chart.yaml — definicja chartu oraz jego metadane i zależności.
Chart.lock — zapis wersji rozwiązanych zależności Helm.
values.yaml — ustawienia wdrożenia Grafany.
templates/tls-secret.yaml — szablon zasobu Kubernetes Secret związanego z TLS.

Katalog packages/prometheus

Zawiera manifesty Kubernetes potrzebne do wdrożenia Prometheusa z pakietu Tanzu.

packageinstall.yaml — definicja instalacji pakietu Prometheus.
values-secret.yaml — konfiguracja przekazywana do pakietu poprzez Kubernetes Secret.
serviceaccount.yaml — konto serwisowe wykorzystywane podczas instalacji.
clusterrole.yaml — definicja uprawnień na poziomie klastra.
clusterrolebinding.yaml — powiązanie uprawnień z odpowiednim podmiotem.
ingress.yaml — konfiguracja dostępu do usługi przez Ingress.
tls-secret.yaml — definicja sekretu związanego z TLS.
kustomization.yaml — lista zasobów zarządzanych przez Kustomize.
3. Instalacja Prometheusa

Prometheus jest instalowany przez mechanizm pakietów Carvel/Tanzu.

Podstawowe parametry z packageinstall.yaml:

Parametr	Wartość
Nazwa instalacji	prometheus
Namespace	vmware-system-tkg
Nazwa pakietu	prometheus.tanzu.vmware.com
Wersja pakietu	2.55.1+vmware.2-tkg.1
Kubernetes Secret z konfiguracją	prometheus-vmware-system-tkg-values
ServiceAccount	prometheus-vmware-system-tkg-sa

Pakiet jest pobierany z repozytorium pakietów skonfigurowanego w środowisku Tanzu. Wersja pakietu jest określona w manifeście packageinstall.yaml.

Przed wdrożeniem należy upewnić się, że klaster ma dostępny odpowiedni pakiet i wymagane zasoby.

Przykładowe sprawdzenie stanu instalacji:

kubectl get packageinstall -n vmware-system-tkg
kubectl describe packageinstall prometheus -n vmware-system-tkg


Aby zastosować manifesty z użyciem Kustomize:

kubectl apply -k packages/prometheus


Przed wykonaniem polecenia należy sprawdzić zawartość kustomization.yaml i upewnić się, że wszystkie wymagane zasoby są przygotowane.

4. Instalacja Grafany

Grafana jest przygotowana jako chart Helm w katalogu HELM/Grafana.

Przed wdrożeniem należy sprawdzić konfigurację:

cat HELM/Grafana/Chart.yaml
cat HELM/Grafana/values.yaml


Sprawdzenie dostępnych zależności:

helm dependency list HELM/Grafana


Przykładowa instalacja lub aktualizacja release'u Grafany:

helm upgrade --install grafana ./HELM/Grafana \
  --namespace <namespace-grafany> \
  --create-namespace


Należy wcześniej ustalić właściwy namespace oraz sprawdzić, czy wartości wymagane w środowisku zostały skonfigurowane w values.yaml.


### Pomocnicze komendy do wygenerowania base64

Grafana:
```sh
base64 < /path/to/grafana.crt | tr -d '\n'
base64 < /path/to/grafana.key | tr -d '\n'
```

Prometheus:
```sh
base64 < /path/to/prometheus.crt | tr -d '\n'
base64 < /path/to/prometheus.key | tr -d '\n'
```

### Alternatywnie: podmiana w klastrze (bez GitOps)

```sh
kubectl -n grafana delete secret grafana-tls-le
kubectl -n grafana create secret tls grafana-tls-le \
  --cert=/path/to/grafana.crt --key=/path/to/grafana.key

kubectl -n tanzu-system-monitoring delete secret prometheus-tls-le
kubectl -n tanzu-system-monitoring create secret tls prometheus-tls-le \
  --cert=/path/to/prometheus.crt --key=/path/to/prometheus.key
```
