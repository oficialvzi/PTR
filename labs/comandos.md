# Guia Rápido de Comandos: VPCS e Linux (Debian/Ubuntu)

- **Instituição:** Universidade de Brasília (UnB)
- **Departamento:** Departamento de Engenharia Elétrica (ENE)
- **Curso:** Engenharia de Redes de Comunicação
- **Disciplina:** ENE0025 - Protocolos de Transporte e Roteamento
- **Professor responsável:** Prof. Dr. Laerte Peotta de Melo
- **Tema:** Referência rápida de comandos do VPCS, do Linux e do Netplan utilizados nos laboratórios

---

## 1. Tabela de comandos do VPCS no PNetLab

| Comando | O que faz | Exemplo |
|---|---|---|
| `ip <ip>/<máscara> <gateway>` | Configura endereço IP e gateway padrão da VPC | `ip 192.168.10.10/24 192.168.10.1` |
| `ip <ip>/<máscara>` | Configura apenas o endereço IP, sem gateway | `ip 192.168.10.10/24` |
| `ip dhcp` | Solicita configuração automática via DHCP | `ip dhcp` |
| `dhcp` | Solicita IP via DHCP | `dhcp` |
| `show` | Exibe a configuração atual da VPC | `show` |
| `ping <ip>` | Testa conectividade com outro host | `ping 192.168.10.1` |
| `ping <ip> -t` | Executa ping contínuo | `ping 192.168.10.1 -t` |
| `trace <ip>` | Mostra o caminho até o destino, semelhante ao traceroute | `trace 8.8.8.8` |
| `arp` | Exibe a tabela ARP da VPC | `arp` |
| `save` | Salva a configuração atual da VPC | `save` |
| `load` | Carrega a configuração salva anteriormente | `load` |
| `history` | Mostra o histórico de comandos digitados | `history` |
| `echo <texto>` | Exibe um texto na tela | `echo teste de rede` |
| `sleep <segundos>` | Aguarda a quantidade de segundos informada | `sleep 5` |
| `help` | Mostra a ajuda de comandos disponíveis | `help` |
| `?` | Mostra ajuda rápida dos comandos | `?` |
| `version` | Exibe a versão do VPCS | `version` |
| `quit` | Encerra a sessão atual | `quit` |
| `rlogin <ip>` | Abre conexão remota simples com outro equipamento, quando suportado | `rlogin 192.168.10.1` |
| `set pcname <nome>` | Define o nome da VPC, quando suportado | `set pcname VPC1` |
| `show ip` | Exibe informações IP da VPC, quando suportado | `show ip` |
| `show arp` | Exibe a tabela ARP, quando suportado | `show arp` |
| `relay` | Exibe ou ajusta parâmetros de relay, em versões que suportam o recurso | `relay` |
| `debug` | Exibe informações de depuração, em versões que suportam esse comando | `debug` |

> Os comandos marcados como "quando suportado" dependem da versão do VPCS instalada no PNetLab; a lista completa de comandos da versão em uso é exibida por `?`.


## 2. Comandos simples de rede no Linux Debian/Ubuntu

| Comando | Descrição |
|---|---|
| `ip a` | Mostra interfaces de rede e endereços IP configurados. |
| `ip link` | Exibe o estado das interfaces de rede. |
| `hostname -I` | Mostra o(s) endereço(s) IP da máquina. |
| `nmcli device status` | Mostra o status das interfaces gerenciadas pelo NetworkManager. |
| `ip route` | Exibe a tabela de roteamento do sistema. |
| `route -n` | Exibe a tabela de roteamento em formato numérico. Requer o pacote `net-tools`. |
| `netstat -rn` | Mostra as rotas configuradas no sistema. Requer o pacote `net-tools`. |
| `sudo ip link set eth0 up` | Ativa a interface `eth0`. |
| `sudo ip link set eth0 down` | Desativa a interface `eth0`. |
| `sudo ip addr add 192.168.1.10/24 dev eth0` | Adiciona um endereço IP manualmente à interface. |
| `sudo ip addr del 192.168.1.10/24 dev eth0` | Remove um endereço IP da interface. |
| `sudo ip route add default via 192.168.1.1` | Define o gateway padrão. |
| `sudo ip route del default` | Remove o gateway padrão. |
| `ping 8.8.8.8` | Testa conectividade com a internet usando um endereço IP. |
| `ping google.com` | Testa conectividade e resolução de nomes via DNS. |
| `cat /etc/resolv.conf` | Exibe os servidores DNS configurados. |
| `sudo systemctl restart NetworkManager` | Reinicia o serviço NetworkManager. |
| `sudo systemctl restart networking` | Reinicia o serviço tradicional de rede (disponível em Debian com `ifupdown`; ausente em instalações padrão do Ubuntu recente, que usa Netplan). |
| `sudo dhclient eth0` | Solicita um endereço IP via DHCP para a interface. |
| `ss -tuln` | Mostra portas abertas e serviços escutando. |
| `ip link show eth0` | Mostra detalhes da interface, incluindo o endereço MAC. |
| `sudo hostnamectl set-hostname novonome` | Altera o nome da máquina. |
| `netplan get` | Mostra a configuração atual do Netplan. |
| `sudo nano /etc/netplan/01-netcfg.yaml` | Abre o arquivo de configuração do Netplan para edição. |
| `sudo netplan apply` | Aplica a configuração definida no Netplan. |
| `which route` | Verifica se o comando `route` está disponível no sistema. |
| `sudo apt update` | Atualiza a lista de pacotes disponíveis. |
| `sudo apt install net-tools` | Instala o pacote `net-tools`, que inclui `route` e `netstat`. |

### 2.1 Observação

| Item | Descrição |
|---|---|
| `ip`, `ip a`, `ip link`, `ip route` | Fazem parte do pacote moderno `iproute2`. |
| `route`, `netstat` | Fazem parte do pacote legado `net-tools`. |
| Recomendação | Em Debian e Ubuntu atuais, recomenda-se priorizar os comandos baseados em `ip`. |


## 3. Netplan no Debian/Ubuntu

### 3.1 O que é o Netplan

Netplan é uma ferramenta de configuração de rede utilizada em distribuições Linux como Ubuntu para definir interfaces, endereços IP, gateway, DNS e DHCP por meio de arquivos em formato YAML.

Em vez de configurar a rede diretamente com comandos isolados, o Netplan permite descrever a configuração de forma declarativa em arquivos localizados normalmente em:

```text
/etc/netplan/
```

---

### 3.2 Estrutura básica

Os arquivos do Netplan utilizam sintaxe YAML. Um exemplo mínimo é:

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
```

#### 3.2.1 Significado dos campos

| Campo | Descrição |
|---|---|
| `network` | Bloco principal da configuração de rede |
| `version: 2` | Versão do formato do Netplan |
| `ethernets` | Define interfaces de rede cabeadas |
| `eth0` | Nome da interface |
| `dhcp4: true` | Habilita obtenção automática de IP via DHCP |

---

### 3.3 Principais comandos

| Comando | Descrição |
|---|---|
| `netplan get` | Mostra a configuração atual do Netplan |
| `sudo nano /etc/netplan/01-netcfg.yaml` | Edita um arquivo de configuração |
| `sudo netplan try` | Aplica temporariamente a configuração para teste |
| `sudo netplan apply` | Aplica definitivamente a configuração |
| `sudo netplan generate` | Gera arquivos intermediários a partir do YAML |

---

### 3.4 Exemplo 1: configuração com DHCP

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
```

#### 3.4.1 Aplicar
```bash
sudo netplan apply
```

---

### 3.5 Exemplo 2: configuração IP estático

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.10/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

#### 3.5.1 Explicação

| Campo | Descrição |
|---|---|
| `dhcp4: false` | Desabilita DHCP IPv4 |
| `addresses` | Define endereço IP manual |
| `192.168.1.10/24` | IP com máscara CIDR |
| `routes` | Define rotas estáticas |
| `to: default` | Define a rota padrão |
| `via: 192.168.1.1` | Define o gateway padrão |
| `nameservers` | Define os servidores DNS |

---

### 3.6 Exemplo 3: configuração com duas interfaces

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 10.0.0.10/24
```

Nesse caso:

- `enp0s3` recebe IP automaticamente
- `enp0s8` recebe IP fixo

---

### 3.7 Exemplo 4: DNS manual

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

---

### 3.8 Exemplo 5: rota estática adicional

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.10.10/24
      routes:
        - to: default
          via: 192.168.10.1
        - to: 172.16.0.0/16
          via: 192.168.10.254
```

#### 3.8.1 Interpretação

| Rota | Função |
|---|---|
| `default via 192.168.10.1` | Saída padrão da máquina |
| `172.16.0.0/16 via 192.168.10.254` | Rota específica para a rede 172.16.0.0/16 |

---

### 3.9 Passo a passo básico

#### 3.9.1 Verificar o nome da interface
```bash
ip a
```

#### 3.9.2 Editar o arquivo do Netplan
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

#### 3.9.3 Validar a indentação do arquivo
YAML é sensível a espaços. Não use tabulação.

#### 3.9.4 Testar a configuração
```bash
sudo netplan try
```

#### 3.9.5 Aplicar definitivamente
```bash
sudo netplan apply
```

---

### 3.10 Cuidados importantes

| Cuidados | Explicação |
|---|---|
| Indentação correta | YAML depende de espaços corretos |
| Nome correto da interface | Use exatamente o nome mostrado por `ip a` |
| Evitar tabs | Use apenas espaços |
| Testar antes de aplicar | `netplan try` ajuda a evitar perda de acesso remoto |
| Backup do arquivo | Recomendável antes de editar |
| Permissões do arquivo | Arquivos de configuração legíveis por outros usuários geram aviso do Netplan; usar `sudo chmod 600 /etc/netplan/*.yaml` |

---

### 3.11 Verificações após aplicar

| Comando | Objetivo |
|---|---|
| `ip a` | Verificar IP configurado |
| `ip route` | Verificar tabela de roteamento |
| `ping 8.8.8.8` | Testar conectividade IP |
| `ping google.com` | Testar conectividade e DNS |
| `cat /etc/resolv.conf` | Verificar resolução de nomes |

---

### 3.12 Exemplo completo comentado

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.0.100/24
      routes:
        - to: default
          via: 192.168.0.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

#### 3.12.1 Resumo da configuração

| Item | Valor |
|---|---|
| Interface | `eth0` |
| IP | `192.168.0.100` |
| Máscara | `/24` |
| Gateway | `192.168.0.1` |
| DNS 1 | `8.8.8.8` |
| DNS 2 | `1.1.1.1` |

---

### 3.13 Erros comuns

| Erro | Possível causa |
|---|---|
| Configuração não aplica | Erro de sintaxe YAML |
| Interface não sobe | Nome da interface incorreto |
| Sem internet | Gateway errado ou ausente |
| Sem resolução de nomes | DNS não configurado corretamente |
| Perda de acesso remoto | Aplicação sem teste prévio |

---

### 3.14 Comandos úteis em laboratório

```bash
ip a
ip route
netplan get
sudo nano /etc/netplan/01-netcfg.yaml
sudo netplan try
sudo netplan apply
ping 8.8.8.8
ping google.com
cat /etc/resolv.conf
```

---

### 3.15 Conclusão

Netplan simplifica a administração de rede em sistemas Debian/Ubuntu ao concentrar a configuração em arquivos YAML claros e organizados. Seu uso é especialmente útil em laboratórios, servidores e ambientes didáticos, pois permite configurar DHCP, IP estático, DNS e rotas de maneira padronizada e reproduzível.

---

## 4. Referências

- Canonical. *Netplan: documentação oficial*. Disponível em: https://netplan.readthedocs.io/.
- Páginas de manual `ip(8)`, `ss(8)` e `netplan(5)`.
- Projeto VPCS. *Virtual PC Simulator*. Disponível em: https://github.com/GNS3/vpcs.
- [**RFC 1918**](https://datatracker.ietf.org/doc/html/rfc1918) - *Address Allocation for Private Internets* (faixas de endereços privados utilizadas nos exemplos).

[Índice Geral (README)](../README.md) | [Laboratório 01 →](lab01.md)


