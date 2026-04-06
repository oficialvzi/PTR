# Comandos simples de rede no Linux Debian/Ubuntu

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
| `sudo systemctl restart networking` | Reinicia o serviço tradicional de rede. |
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

## Observação

| Item | Descrição |
|---|---|
| `ip`, `ip a`, `ip link`, `ip route` | Fazem parte do pacote moderno `iproute2`. |
| `route`, `netstat` | Fazem parte do pacote legado `net-tools`. |
| Recomendação | Em Debian e Ubuntu atuais, recomenda-se priorizar os comandos baseados em `ip`. |
