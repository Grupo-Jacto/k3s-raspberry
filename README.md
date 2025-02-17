# Cluster Raspberry Pi

## Descrição

Este projeto tem como objetivo criar um cluster de Raspberry Pi com [K3S](https://k3s.io/).

## Configuração

### Instalar o sistema operacional

Download da última versão do Raspbian no [site oficial](https://www.raspberrypi.org/downloads/raspbian/). Siga as instruções para instalar a imagem no cartão SD.

#### Versão SO

Utilize a versão `Raspberry PI OS Lite`.

#### Gerais

Defina configurações gerais:

- `hostname`: `k3s-master` para o principal e `k3s-worker-1`, `k3s-worker-2`, ... para os demais.
- `username`: `raspberry`.
- `password`: Crie uma senha segura.

Caso utilize a WiFi para configurações iniciais das Raspberry, defina os dados abaixo:

- `SSID`: Nome da rede WiFi.
- `PASSWORD`: Senha da rede WiFi.
- `country`: Código do país.

Caso queira definir configurações de região, idioma e teclado para o Brasil, siga as instruções abaixo:

- `Timezone`: `America/Sao_Paulo`.
- `Keyboard Layout`: `br`.

#### Serviços

Ative SSH, selecione `Autenticar via senha` ou `Autenticar via chave` e coloque a chave pública.

#### Opções

Desabilite a Telemetria.

### Acessar via SSH

Para acessar os Raspberry Pi via SSH utilize o comando abaixo:

#### Acesso

Acessar a Raspberry Pi, substitua `<KEY_RSA>` pelo arquivo de chave privada e `<IP_ADDRESS>` pelo IP do Raspberry Pi:

```sh
ssh -i <KEY_RSA> raspberry@<IP_ADDRESS>
```

### Configurar os IPs estáticos

Caso o arquivo `/etc/dhcpcd.conf` não exista execute os comandos de [configuração do cliente DHCP](#erro-de-configuração-do-cliente-dhcp). E prossiga com as instruções abaixo.

#### Editar o arquivo de configuração do cliente DHCP

```sh
sudo nano /etc/dhcpcd.conf
```

#### Adicionar as configurações de IP estático

Cenário com dois Raspberry Pi conectados diretamente:

**Raspberry Pi - Master:**

```conf
# Configuração para eth0
interface eth0
static ip_address=192.168.1.2/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8
metric 300

# Configuração para wlan0
interface wlan0
dhcp
metric 200
```

**Raspberry Pi - Worker 1:**

```conf
# Configuração para eth0
interface eth0
static ip_address=192.168.1.3/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1 8.8.8.8
metric 300

# Configuração para wlan0
interface wlan0
dhcp
metric 200
```

Caso utilize um roteador substitua `static routers=192.168.1.1` pelo IP do roteador. Caso a conexão seja via WiFi substitua `eth0` por `wlan0`.

#### Reiniciar o cliente DHCP e Raspberry Pi

```sh
sudo systemctl restart dhcpcd
```

```sh
sudo reboot
```

### Instalar o K3S

Caso ocorra erro de memória ao tentar instalar o K3S, siga as instruções de [erro de memória](#erro-de-memória).

#### Instalar no Raspberry Pi - Master

```sh
curl -sfL https://get.k3s.io | sh -
```

#### Obter o token de acesso

```sh
sudo cat /var/lib/rancher/k3s/server/node-token
```

#### Instalar nos Raspberry Pi - Workers

```sh
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.2:6443 K3S_TOKEN=<TOKEN> sh -
```

Substitua `<TOKEN>` pelo token obtido no Raspberry Pi - Master.

#### Verificar o status do K3S

```sh
sudo systemctl status k3s
```

```sh
kubectl get nodes
```

Caso gere erro de permissão, [configure a permissão.](#erro-de-status-do-k3s)

### Erro de configuração do cliente DHCP

Caso não tenha o arquivo `/etc/dhcpcd.conf` siga as instruções abaixo.

#### Atualizar o sistema

```sh
sudo apt-get update
sudo apt-get upgrade
```

#### Instalar o cliente DHCP

```sh
sudo apt-get install dhcpcd5
```

#### Habilitar o cliente DHCP

```sh
sudo systemctl enable dhcpcd
```

#### Iniciar o cliente DHCP

```sh
sudo systemctl start dhcpcd
```

#### Verificar o status do cliente DHCP

```sh
sudo systemctl status dhcpcd
```

#### Continue com as [Configurações os IPs estáticos.](#configurar-os-ips-estáticos)

### Erro de memória

Caso ocorra o erro abaixo:

```text
"failed to find memory cgroup, you may need to add \"cgroup_memory=1 cgroup_enable=memory\" to your linux cmdline (/boot/cmdline.txt on a Raspberry Pi)"
```

#### Opção 1

Adicione as configurações de memoria.

##### Configuração de WiFi

**Edite o arquivo:**

```sh
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
```

**Adicione as configurações de Wifi:**

```text
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=BR

network={
  ssid="SSID"
  psk="PASSWORD"
  key_mgmt=WPA-PSK
}
```

Ajuste as configurações de `SSID` e `PASSWORD` conforme a rede WiFi.

##### Configuração de memória

**Edite o arquivo:**

```sh
sudo nano /boot/firmware/cmdline.txt
```

**Adicione o texto ao final do arquivo:**

```text
cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
```

**Reinicie o Raspberry Pi:**

```sh
sudo reboot
```

#### Opção 2

Atualize o firmware:

```sh
sudo PRUNE_MODULES=1 RPI_REBOOT=1 SKIP_WARNING=1 rpi-update
```

#### Continue com a [Instalação do K3S.](#instalar-o-k3s)

### Erro de status do K3S

Configure a permissão:

#### Configurar a Variável de Ambiente KUBECONFIG

Defina a variável de ambiente KUBECONFIG para apontar para um arquivo de configuração local:

```sh
export KUBECONFIG=~/.kube/config
```

#### Criar um Arquivo de Configuração LocalCrie uma cópia do arquivo de configuração do K3S que possa ser usada com os comandos `kubectl`:

```sh
mkdir -p ~/.kube
sudo k3s kubectl config view --raw > "$KUBECONFIG"
chmod 600 "$KUBECONFIG"
```

O comando `chmod 600` garante que o arquivo seja legível e gravável apenas pelo proprietário.

#### Tornar a Configuração PersistentePara garantir que a variável `KUBECONFIG` seja persistente após reiniciar o terminal ou o sistema, adicione a linha abaixo ao seu arquivo `~/.bashrc` ou `~/.profile`:

```sh
echo 'export KUBECONFIG=~/.kube/config' >> ~/.bashrc
source ~/.bashrc
```

#### Verificar a ConfiguraçãoExecute o comando abaixo para verificar se a configuração está funcionando corretamente:

```sh
kubectl get nodes
```

#### Continue com a [Verificação do status do K3S.](#verificar-o-status-do-k3s)
