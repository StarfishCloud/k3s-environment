# K3S environment

Repositório utilizado para criar máquinas virtuais que serão utilizadas para levantar o K3S.


## Requisitos necessários
* Virtualbox
* vagrant

## Criando máquinas virtuais

```sh
$ vagrant up
```

Aguardar o download da imagem base, a criação das VMs e a instalação das dependências. Se durante o provisionamento da infraestrutura o terminal perguntar qual a interface que será utilizado para se criar a bridge, escolha o número referente a interface que prover acesso a rede externa (internet). Ex:

```
==> k3s-master: Available bridged network interfaces:
1) wlx0013ef4102cb
2) eno1
3) br-kub
4) virbr0
5) lxcbr0
6) br-4a0ba2870b03
7) docker0
==> k3s-master: When choosing an interface, it is usually the one that is
==> k3s-master: being used to connect to the internet.
==> k3s-master: 
    k3s-master: Which interface should the network bridge to? 1
```

Após isso, verificar o status do ambiente criado com o comando `vagrant status`.
```sh
$ vagrant status          
Current machine states:

k3s-master01              running (virtualbox)
k3s-master02              running (virtualbox)
k3s-master03              running (virtualbox)
k3s-worker01              running (virtualbox)
k3s-worker02              running (virtualbox)
postgres                  running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

Para acessar uma VM específica, utilizar o comando `vagrant ssh`.

```sh
$ vagrant ssh k3s-master01
```

Caso deseje parar as VMs, utilize o comando `vagrant shutdown`.

Para destruir o ambiente (apagar as VMs) execute o comando: `vagrant destroy -f`.

## Instalação do k3s

### Configuração do BD externo

Por padrão, o k3s levanta o SQlite como banco de dados dos estado do cluster. Entretanto este banco não suporta múltiplos servidores (masters). Para tal, é necessário configurar um BD externo e apontá-lo para ser utilizado pelo cluster. Uma estratégia que simplifica a complexidade do cluster é levantar um único BD e apontá-lo no master.

É possível também levantar o k3s com o etcd integrado. Contudo, a documentação não recomenda pois degrada o sdcard dos dispositivos, derrubando consideravelmente a performance do cluster.

No Vagrantfile, já estão definidos as instruções de instalação e configuração do PostgreSQL.

### Master
Acesse a máquina virtual:
```sh
$ vagrant ssh k3s-master01
(k3s-master01)$ curl -sfL https://get.k3s.io | sh -s - server --datastore-endpoint="postgres://k3s_user:k3s_password@192.168.56.50:5432/kubernetes" --tls-san=192.168.56.101
```

O comando acima cria um master sem nenhum worker. Para configurar os workers e os demais masters, é necessário inicialmente obter o token gerado para identificação do cluster. Consulte o arquivo `/var/lib/rancher/k3s/server/token` e copie o token.

```sh
(k3s-master01) $ sudo cat /var/lib/rancher/k3s/server/token
K1038f5d9a8398f2beffbd408323750eacbabf8b0743d5b88bffbae76fe443cbc32::server:2dad61079c0393943aa51b27443313eb
```

Para adicionar os demais masters, acesse-os e execute:
```sh
$ vagrant ssh k3s-master02
(k3s-master02) $ curl -sfL https://get.k3s.io | sh -s - server --token=K1038f5d9a8398f2beffbd408323750eacbabf8b0743d5b88bffbae76fe443cbc32::server:2dad61079c0393943aa51b27443313eb --datastore-endpoint="postgres://k3s_user:k3s_password@192.168.56.50:5432/kubernetes"

...
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
(k3s-master02) $ exit

$ vagrant ssh k3s-master03
(k3s-master03) $ curl -sfL https://get.k3s.io | sh -s - server --token=K1038f5d9a8398f2beffbd408323750eacbabf8b0743d5b88bffbae76fe443cbc32::server:2dad61079c0393943aa51b27443313eb --datastore-endpoint="postgres://k3s_user:k3s_password@192.168.56.50:5432/kubernetes"
...
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
(k3s-master03) $ exit
```
### Workers
Verifique se a porta do kube-api já está levantada
```
$ vagrant ssh k3s-worker01
(worker-01) $ nc 192.168.56.101 6443
```

Em todos os workers, execute o comando:
```sh
(k3s-worker-) $ curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent" K3S_TOKEN="K1038f5d9a8398f2beffbd408323750eacbabf8b0743d5b88bffbae76fe443cbc32::server:2dad61079c0393943aa51b27443313eb" sh -s - --server https://192.168.56.101:6443
...
[INFO]  env: Creating environment file /etc/systemd/system/k3s-agent.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s-agent.service
[INFO]  systemd: Enabling k3s-agent unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s-agent.service → /etc/systemd/system/k3s-agent.service.
[INFO]  systemd: Starting k3s-agent
(k3s-worker-)
```

Acesse a VM k3s-master01 e execute:
```sh
$ vagrant ssh k3s-master01
(k3s-master01) $ sudo k3s kubectl get nodes
NAME           STATUS   ROLES                  AGE     VERSION
k3s-master01   Ready    control-plane,master   10m     v1.31.4+k3s1
k3s-master02   Ready    control-plane,master   8m46s   v1.31.4+k3s1
k3s-master03   Ready    control-plane,master   7m47s   v1.31.4+k3s1
k3s-worker01   Ready    <none>                 6m27s   v1.31.4+k3s1
k3s-worker02   Ready    <none>                 5m26s   v1.31.4+k3s1
```

Pronto, seu cluster k3s foi iniciado com sucesso. Enjoy it!

## Levantando container de exemplo no ambiente
Acesse novamente a VM k3s-master01

Crie um arquivo chamado app.yml
Arquivo app.yml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: starfishcloud-website
spec:
  replicas: 1
  selector:
    matchLabels:
      app: starfishcloud-website
  template:
    metadata:
      labels:
        app: starfishcloud-website
    spec:
      containers:
        - name: starfishcloud-website-ct
          image: lesilva00/starfish-website:v1.0
          ports:
            - containerPort: 3000
```