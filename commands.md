## Comandos úteis para gerenciamento de Kubernetes

### Quando se deseja atualizar a imagem de algum deployment
```sh
kubectl set image deployment/<deployment_name> <container_name>=<image_name> -n namespace
```
Isso faz com que a imagem seja atualizada e um redeploy seja realizado na sequência
___

### Para atualizar a imagem de um conjunto de deployments que tenham o nome que começa com o mesmo prefíxo.

Por exemplo, suponha que sua aplicação seja `application` com os seguintes sufixos:

application-web

application-server

application-client

```sh
for deployment_name in $(kubectl get deployments -o jsonpath='{.items[*].metadata.name}' | tr ' ' '\n' | grep '^<applicaiton>'); do
  container_name=$deployment
  image_name=<image_name>
  kubectl set image deployment/$deployment_name $container_name=$image_name
done
```

Acima fazemos um container_name=$deployment pois o nome do nosso container é igual ao nome do deployment. Essa substituição é meramente para não causar confusão, mas você pode modificar como desejar, especialmente se teu container possuir um nome diferente do deployment.

Substitua também o valor da variável image_name.
___
