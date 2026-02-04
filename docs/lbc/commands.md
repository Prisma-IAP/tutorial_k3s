# Comandos básicos para gerenciar os deploys no K3s

O tutorial a seguir tem como objetivo auxiliar com instruções para gerenciar aplicações implantadas (deploys) no cluster com K3s

- Remover um deployment realizado: Em arquivos com múltiplas implementações (Deployment, Service, Ingress, etc...) com o commando abaixo é possível remover tudo o que foi implementado.
```shell
kubectl delete -f file_name.yaml
```