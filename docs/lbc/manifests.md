# Fazendo o deploy de aplicações e serviços no K3s

Nesta etapa do tutorial, é importante que você já tenha o K3s instalado e configurado. Em seguida, basta seguir as etapas para a implementação dos serviços desejados.

Para utilizar o arquivo ***pandaweb.yaml***, basta aplicar as configurações com um dos comandos abaixo:
```shell
kubectl create -f pandaweb.yaml --save-config
```

ou

```shell
kubectl apply -f pandaweb.yaml
```
A única diferença entre os comandos é que o segundo gera uma notificação referente as configurações realizadas durante o deploy.

--- 

Para utilizar o arquivo ***rancherdemo.yaml***, basta seguir o mesmo passo acima. Caso deseje alterar alguma informação na seção demonstrada abaixo, basta seguir as configurações que serão apresentadas após o trecho de código.

```yaml
env:
- name: CONTAINER_COLOR
value: orange
- name: PETS
value: chameleons
- name: TITLE
value: TCC Load Balancer Demo
```

- No parâmetro ***CONTAINER_COLOR***, pode adicionar as seguintes cores:
    - red
    - orange
    - yellow
    - olive
    - green
    - teal
    - blue
    - violet
    - purple
    - pink
    - black

- No parâmetro ***PETS***, pode adicionar os seguintes animais:
    - cows
    - chameleons
    - cowmeleons

- Já no parâmetro ***TITLE***, basta adicionar o texto desejado.

Para visualizar outros possíveis parâmetros que podem ser adicionados, basta acessar este [repositório](https://github.com/ronaldomendes/rancher-demo)