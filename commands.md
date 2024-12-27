## Comandos úteis para gerenciamento de Kubernetes

### Quando se deseja atualizar a imagem de algum deployment
```sh
kubectl set image deployment/<deployment_name> <container_name>=<image_name> -n namespace
```
Isso faz com que a imagem seja atualizada e um redeploy seja realizado na sequência
___
