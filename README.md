# Loki

Este documento apresenta o passo-a-passo para instalação do loki no grafana.

Caso você já possua o prometheus e grafana instalado, pule para o passo de configuração/instalação do loki.

> Todo o processo deve ser realizado no terminal e a pasta atual deve ser o root do projeto.
>
> A execução do processo é todo executando em ambiente kubernetes com o helm chart

## Passo 1: Instalação do grafana

Execute o comando: `helm upgrade --install -n metrics grafana grafana/grafana -f ./values-grafana.yaml`

## Passo 2: Instalação do prometheus

Execute o comando: `helm upgrade --install -n metrics prometheus prometheus-community/prometheus -f ./values-prometheus.yaml`

## Passo 3: Instalação do eventrouter

Execute o comando: `helm upgrade --install -n metrics eventrouter stable/eventrouter -f ./values-eventrouter.yaml`

## Passo 4: Instalação do loki

Execute o comando: `helm upgrade --install -n metrics loki loki/loki-stack -f ./values-loki.yaml`

## Passo 5: Adicionando datasources

Logado no grafana, acesse no menu lateral ***Configuration > Data Sources > Add data source*** e adicione 3 fontes de dados:

1. Loki:
    - Fonte: Loki
    - Nome: Loki
    - URL: http://loki:3100
2. Prometheus:
    - Fonte: Prometheus
    - Nome: Prometheus
    - URL: http://prometheus-server
3. Loki-Prometheus:
    - Fonte: Prometheus
    - Nome: Loki-Prometheus
    - URL: http://loki:3100/loki

## Passo 6: Criando dashboard

Acesso no menu lateral ***Create > Dashboard***.

Na tela do novo dashboard, vá para o menu superior, acesse ***Dashboard settings > Variables*** e adicione 2 variáveis:

1. **$namespace**:
    - nome: namespace
    - tipo: Query
    - Fonte: Prometheus
    - Atualização: On Dashboard Load
    - Busca: `label_values(kube_pod_info{namespace=~"dev|test"}, namespace)`
2. **$container**:
    - nome: container
    - tipo: Query
    - Fonte: Prometheus
    - Atualização: On Dashboard Load
    - Busca: `label_values(kube_pod_container_info{namespace="$namespace"}, container)`

Agora, salve as informações e crie um painel de escolher vizualização, escolha o tipo de vizualização `Logs` e no data source da vizualização, escolha a Query `Loki`.

Após esta etapa, adicione a seguinte query: `{app=~"eventrouter|.*", namespace="$namespace", container="$container"}`.

Pronto. Seu Dashboard exibe os logs do loki com os dados de event do kubernetes.
