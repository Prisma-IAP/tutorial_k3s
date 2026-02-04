# Configuração básica do Traefik Dashboard

Para realizar a configuração do ***Traefik Dashboard*** é necessário manter a instalação original do ***K3S***, sem remover o Traefik.

Em seguida, crie um arquivo de configuração no seguinte caminho: ``/var/lib/rancher/k3s/server/manifests/traefik-config.yaml``.

Com o arquivo criado, adicione o seguinte conteúdo:
```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: traefik
  namespace: kube-system
spec:
  valuesContent: |-
    dashboard:
      enabled: true
    ports:
      traefik:
        expose: true
    logs:
      access:
        enabled: true
```

Após salvar o arquivo, acesse em outro computador na mesma rede o endereço do seu servidor ***K3S*** conforme o exemplo a seguir: 

http://192.168.0.80:9000/dashboard/