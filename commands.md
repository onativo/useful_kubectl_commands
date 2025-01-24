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
