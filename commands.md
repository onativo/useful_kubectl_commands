## Comandos úteis para gerenciamento de Kubernetes

### Para atualizar a imagem de um único deployment
```sh
kubectl set image deployment/<deployment_name> <container_name>=<image_name> -n namespace
```
| variável | info
|-------|-------
|`deployment_name`| troque pelo nome do deployment|
|`namespace`| insira o nome do namespace alvo|
|`image_name`| substitua pelo nome da imagem|

Isso faz com que a imagem seja atualizada e um redeploy seja realizado na sequência
___

### Para atualizar a imagem de um conjunto de deployments que tenham o mesmo prefixo (ou mesmo nome)

Por exemplo, suponha que sua aplicação seja `application` com os seguintes sufixos:

application-web

application-server

application-client

```sh
for deployment_name in $(kubectl get deployments -n namespace -o jsonpath='{.items[*].metadata.name}' | tr ' ' '\n' | grep '^app-'); do container_name=$deployment_name; image_name=image_name; kubectl set image deployment/$deployment_name $container_name=$image_name -n namespace; done
```
| variável | info
|-------|-------
|`app-`| troque pelo nome do deployment|
|`namespace`| insira o nome do namespace alvo|
|`image_name`| substitua pelo nome da imagem|

Acima fazemos um container_name=$deployment pois o nome do nosso container é igual ao nome do deployment. Essa substituição é meramente para não causar confusão, mas você pode modificar como desejar, especialmente se teu container possuir um nome diferente do deployment.

Substitua também o valor da variável image_name.
___

### Para modificar o número de replicas de um determinado deployment num certo namespace

```sh
for deployment_name in $(kubectl get deployments -n <namespace> -o jsonpath='{.items[*].metadata.name}' | tr ' ' '\n' | grep '^app-'); do if [ $(kubectl get deployment $deployment_name -n <namespace> -o jsonpath='{.status.replicas}') -gt 0 ]; then kubectl scale deployment/$deployment_name --replicas=0 -n <namespace>; fi; done
```
| variável | info
|-------|-------
|`app-`| troque pelo nome do deployment|
|`namespace`| insira o nome do namespace alvo|
|`--replicas=0`| substitua para quanto desejar|
___

### Para visualizar logs de um pod

```sh
kubectl logs <pod_name> -n <namespace>
```

Para seguir os logs em tempo real (similar ao `tail -f`):
```sh
kubectl logs -f <pod_name> -n <namespace>
```

Para ver logs de um container específico em um pod com múltiplos containers:
```sh
kubectl logs <pod_name> -c <container_name> -n <namespace>
```

Para ver logs dos últimos N minutos:
```sh
kubectl logs <pod_name> --since=10m -n <namespace>
```
| variável | info
|-------|-------
|`pod_name`| nome do pod|
|`namespace`| nome do namespace|
|`container_name`| nome do container (se houver múltiplos)|
|`--since=10m`| logs desde os últimos 10 minutos (pode usar `1h`, `30s`, etc)|
___

### Para visualizar logs de todos os pods de um deployment

```sh
kubectl logs -f deployment/<deployment_name> -n <namespace>
```

Para ver logs de todos os pods com um label específico:
```sh
kubectl logs -f -l app=<app_name> -n <namespace>
```
| variável | info
|-------|-------
|`deployment_name`| nome do deployment|
|`namespace`| nome do namespace|
|`app_name`| valor do label `app`|
___

### Para fazer port-forward e acessar um serviço localmente

```sh
kubectl port-forward <resource_type>/<resource_name> <local_port>:<pod_port> -n <namespace>
```

Exemplos:
```sh
# Port-forward de um pod
kubectl port-forward pod/<pod_name> 8080:80 -n <namespace>

# Port-forward de um deployment
kubectl port-forward deployment/<deployment_name> 8080:80 -n <namespace>

# Port-forward de um service
kubectl port-forward svc/<service_name> 8080:80 -n <namespace>
```
| variável | info
|-------|-------
|`resource_type`| tipo do recurso (pod, deployment, svc)|
|`resource_name`| nome do recurso|
|`local_port`| porta local que será usada|
|`pod_port`| porta do pod/serviço no cluster|
|`namespace`| nome do namespace|
___

### Para executar comandos dentro de um pod

```sh
kubectl exec -it <pod_name> -n <namespace> -- <comando>
```

Exemplos:
```sh
# Abrir shell interativo
kubectl exec -it <pod_name> -n <namespace> -- /bin/bash

# Executar comando específico
kubectl exec <pod_name> -n <namespace> -- ls -la /app

# Executar em container específico (pod com múltiplos containers)
kubectl exec -it <pod_name> -c <container_name> -n <namespace> -- /bin/bash
```
| variável | info
|-------|-------
|`pod_name`| nome do pod|
|`namespace`| nome do namespace|
|`container_name`| nome do container (se houver múltiplos)|
|`comando`| comando a ser executado|
___

### Para descrever recursos e ver detalhes

```sh
kubectl describe <resource_type>/<resource_name> -n <namespace>
```

Exemplos:
```sh
# Descrever um pod
kubectl describe pod <pod_name> -n <namespace>

# Descrever um deployment
kubectl describe deployment <deployment_name> -n <namespace>

# Descrever um service
kubectl describe svc <service_name> -n <namespace>

# Descrever um ingress
kubectl describe ingress <ingress_name> -n <namespace>
```
| variável | info
|-------|-------
|`resource_type`| tipo do recurso (pod, deployment, svc, ingress, etc)|
|`resource_name`| nome do recurso|
|`namespace`| nome do namespace|
___

### Para listar recursos com filtros e formatação

```sh
# Listar pods com labels específicos
kubectl get pods -l app=<app_name> -n <namespace>

# Listar pods em todos os namespaces
kubectl get pods --all-namespaces

# Listar pods com formatação customizada
kubectl get pods -o wide -n <namespace>

# Listar pods em formato JSON
kubectl get pods -o json -n <namespace>

# Listar pods em formato YAML
kubectl get pods -o yaml -n <namespace>

# Listar pods com informações específicas (custom columns)
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName -n <namespace>
```
| variável | info
|-------|-------
|`app_name`| valor do label para filtrar|
|`namespace`| nome do namespace|
___

### Para ver eventos do namespace

```sh
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

Para ver eventos de um recurso específico:
```sh
kubectl get events --field-selector involvedObject.name=<pod_name> -n <namespace>
```
| variável | info
|-------|-------
|`namespace`| nome do namespace|
|`pod_name`| nome do pod (ou outro recurso)|
___

### Para verificar o status e health dos pods

```sh
# Ver status de todos os pods
kubectl get pods -n <namespace>

# Ver pods que não estão rodando
kubectl get pods --field-selector=status.phase!=Running -n <namespace>

# Ver pods que estão com erro
kubectl get pods -n <namespace> | grep -v Running

# Verificar readiness e liveness
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.conditions[?(@.type=="Ready")].status}{"\n"}{end}' -n <namespace>
```
| variável | info
|-------|-------
|`namespace`| nome do namespace|
___

### Para fazer rollout e gerenciar deployments

```sh
# Ver histórico de rollouts
kubectl rollout history deployment/<deployment_name> -n <namespace>

# Ver status do rollout
kubectl rollout status deployment/<deployment_name> -n <namespace>

# Fazer rollback para versão anterior
kubectl rollout undo deployment/<deployment_name> -n <namespace>

# Fazer rollback para versão específica
kubectl rollout undo deployment/<deployment_name> --to-revision=<revision_number> -n <namespace>

# Pausar um rollout
kubectl rollout pause deployment/<deployment_name> -n <namespace>

# Retomar um rollout pausado
kubectl rollout resume deployment/<deployment_name> -n <namespace>
```
| variável | info
|-------|-------
|`deployment_name`| nome do deployment|
|`namespace`| nome do namespace|
|`revision_number`| número da revisão (veja no histórico)|
___

### Para deletar recursos

```sh
# Deletar um pod
kubectl delete pod <pod_name> -n <namespace>

# Deletar um deployment
kubectl delete deployment <deployment_name> -n <namespace>

# Deletar todos os pods com um label específico
kubectl delete pods -l app=<app_name> -n <namespace>

# Deletar todos os recursos de um namespace (CUIDADO!)
kubectl delete all --all -n <namespace>

# Deletar com grace period (tempo para encerrar graciosamente)
kubectl delete pod <pod_name> --grace-period=30 -n <namespace>

# Forçar deleção imediata
kubectl delete pod <pod_name> --force --grace-period=0 -n <namespace>
```
| variável | info
|-------|-------
|`pod_name`| nome do pod|
|`deployment_name`| nome do deployment|
|`app_name`| valor do label|
|`namespace`| nome do namespace|
___

### Para aplicar e atualizar recursos

```sh
# Aplicar um arquivo YAML
kubectl apply -f <arquivo.yaml>

# Aplicar todos os arquivos YAML de um diretório
kubectl apply -f <diretorio>/

# Aplicar e ver o que será criado/modificado (dry-run)
kubectl apply -f <arquivo.yaml> --dry-run=client -o yaml

# Substituir recursos (deleta e recria)
kubectl replace -f <arquivo.yaml>

# Criar recursos a partir de um arquivo
kubectl create -f <arquivo.yaml>
```
| variável | info
|-------|-------
|`arquivo.yaml`| caminho para o arquivo YAML|
|`diretorio`| caminho para o diretório com arquivos YAML|
___

### Para copiar arquivos de/para pods

```sh
# Copiar arquivo do pod para local
kubectl cp <namespace>/<pod_name>:<caminho_no_pod> <caminho_local>

# Copiar arquivo local para o pod
kubectl cp <caminho_local> <namespace>/<pod_name>:<caminho_no_pod>

# Copiar diretório
kubectl cp <namespace>/<pod_name>:<diretorio_no_pod> <diretorio_local> -c <container_name>
```
| variável | info
|-------|-------
|`namespace`| nome do namespace|
|`pod_name`| nome do pod|
|`caminho_no_pod`| caminho do arquivo dentro do pod|
|`caminho_local`| caminho do arquivo no seu computador|
|`container_name`| nome do container (se houver múltiplos)|
___

### Para ver recursos e suas relações

```sh
# Ver todos os recursos em um namespace
kubectl get all -n <namespace>

# Ver recursos com suas labels
kubectl get pods --show-labels -n <namespace>

# Ver recursos com seus selectors
kubectl get svc -o wide -n <namespace>

# Ver endpoints de um service
kubectl get endpoints <service_name> -n <namespace>

# Ver configurações e secrets montados em pods
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.volumes[*].configMap.name}{"\t"}{.spec.volumes[*].secret.secretName}{"\n"}{end}' -n <namespace>
```
| variável | info
|-------|-------
|`namespace`| nome do namespace|
|`service_name`| nome do service|
___
