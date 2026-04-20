# Laboratório 05 – Roteamento Dinâmico com RIP e OSPF

**Disciplina:** ENE0025 - Protocolos de Transporte e Roteamento  
**Professor responsável:** Prof. Dr. Laerte Peotta de Melo  
**Monitores:** Victor Lima dos Santos / Beatriz Silva Nascimento

---

## 1. Objetivo

Configurar e validar o roteamento dinâmico em uma topologia com três roteadores, comparando o funcionamento dos protocolos **RIP** e **OSPF** em um mesmo cenário.

---

## 2. Objetivos específicos

Ao final deste laboratório, o estudante deverá ser capaz de:

- configurar interfaces IP em roteadores Cisco;
- ativar roteamento dinâmico com RIP;
- ativar roteamento dinâmico com OSPF;
- verificar tabelas de roteamento;
- validar conectividade entre redes remotas;
- comparar as características básicas de RIP e OSPF.

---

## 3. Fundamentação teórica resumida

O **RIP** é um protocolo de roteamento do tipo **distance-vector**, que utiliza **contagem de saltos** como métrica, com limite de **15 hops**, sendo adequado para cenários menores e mais simples. O material-base do curso apresenta o RIP como parte fundamental da introdução ao roteamento dinâmico.

O **OSPF** é um protocolo de roteamento do tipo **link-state**, baseado no algoritmo **SPF/Dijkstra**, com maior capacidade de escalabilidade, convergência mais eficiente e melhor organização lógica da rede. O plano de ensino da disciplina também destaca RIP e OSPF como tópicos centrais dos laboratórios em ambiente emulado.

---

## 4. Topologia do laboratório

A topologia possui três roteadores interligando três unidades:

- **Router-RJ** — Rio de Janeiro
- **Router-SP** — São Paulo
- **Router-BH** — Belo Horizonte

Cada unidade possui duas LANs locais, e os roteadores são interligados por duas redes WAN.

```mermaid
flowchart LR
  %% =========================
  %% WAN entre roteadores
  %% =========================
  RJR["Router-RJ<br/>Rio de Janeiro"]
  SPR["Router-SP<br/>São Paulo"]
  BHR["Router-BH<br/>Belo Horizonte"]

  RJR ---|WAN 172.16.100.0/24| SPR
  SPR ---|WAN 172.16.200.0/24| BHR

  %% =========================
  %% Rio de Janeiro
  %% =========================
  subgraph RJ["Rio de Janeiro"]
    direction TB
    RJR
    SWRJ10["SW-RJ-10<br/>2960-24TT<br/>172.16.10.0/24"]
    SWRJ20["SW-RJ-20<br/>2960-24TT<br/>172.16.20.0/24"]

    RJR --- SWRJ10
    RJR --- SWRJ20

    PCRJ101["PC-RJ-10-1<br/>172.16.10.1"]
    PCRJ102["PC-RJ-10-2<br/>172.16.10.2"]
    PCRJ201["PC-RJ-20-1<br/>172.16.20.1"]
    PCRJ202["PC-RJ-20-2<br/>172.16.20.2"]

    SWRJ10 --- PCRJ101
    SWRJ10 --- PCRJ102
    SWRJ20 --- PCRJ201
    SWRJ20 --- PCRJ202
  end

  %% =========================
  %% São Paulo
  %% =========================
  subgraph SP["São Paulo"]
    direction TB
    SPR
    SWSP30["SW-SP-30<br/>2960-24TT<br/>172.16.30.0/24"]
    SWSP40["SW-SP-40<br/>2960-24TT<br/>172.16.40.0/24"]

    SPR --- SWSP30
    SPR --- SWSP40

    PCSP301["PC-SP-30-1<br/>172.16.30.1"]
    PCSP302["PC-SP-30-2<br/>172.16.30.2"]
    PCSP401["PC-SP-40-1<br/>172.16.40.1"]
    PCSP402["PC-SP-40-2<br/>172.16.40.2"]

    SWSP30 --- PCSP301
    SWSP30 --- PCSP302
    SWSP40 --- PCSP401
    SWSP40 --- PCSP402
  end

  %% =========================
  %% Belo Horizonte
  %% =========================
  subgraph BH["Belo Horizonte"]
    direction TB
    BHR
    SWBH50["SW-BH-50<br/>2960-24TT<br/>172.16.50.0/24"]
    SWBH60["SW-BH-60<br/>2960-24TT<br/>172.16.60.0/24"]

    BHR --- SWBH50
    BHR --- SWBH60

    PCBH501["PC-BH-50-1<br/>172.16.50.1"]
    PCBH502["PC-BH-50-2<br/>172.16.50.2"]
    PCBH601["PC-BH-60-1<br/>172.16.60.1"]
    PCBH602["PC-BH-60-2<br/>172.16.60.2"]

    SWBH50 --- PCBH501
    SWBH50 --- PCBH502
    SWBH60 --- PCBH601
    SWBH60 --- PCBH602
  end

  %% =========================
  %% Cores
  %% =========================
  classDef router fill:#dbeafe,stroke:#1d4ed8,color:#111827,stroke-width:2px;
  classDef switch fill:#dcfce7,stroke:#16a34a,color:#111827,stroke-width:2px;
  classDef host fill:#fef3c7,stroke:#d97706,color:#111827,stroke-width:1.5px;
  classDef wan fill:#fee2e2,stroke:#dc2626,color:#111827,stroke-width:2px;

  class RJR,SPR,BHR router;
  class SWRJ10,SWRJ20,SWSP30,SWSP40,SWBH50,SWBH60 switch;
  class PCRJ101,PCRJ102,PCRJ201,PCRJ202,PCSP301,PCSP302,PCSP401,PCSP402,PCBH501,PCBH502,PCBH601,PCBH602 host;

```

---

## 5. Plano de endereçamento

O cenário segue uma lógica simples com redes da faixa `172.16.0.0/16`, organizadas em sub-redes `/24`, o que facilita a configuração inicial e a visualização das redes.

### 5.1 Redes utilizadas

| Segmento | Rede | Máscara |
|---|---|---|
| LAN RJ-10 | 172.16.10.0 | 255.255.255.0 |
| LAN RJ-20 | 172.16.20.0 | 255.255.255.0 |
| WAN RJ-SP | 172.16.100.0 | 255.255.255.0 |
| LAN SP-30 | 172.16.30.0 | 255.255.255.0 |
| LAN SP-40 | 172.16.40.0 | 255.255.255.0 |
| WAN SP-BH | 172.16.200.0 | 255.255.255.0 |
| LAN BH-50 | 172.16.50.0 | 255.255.255.0 |
| LAN BH-60 | 172.16.60.0 | 255.255.255.0 |

### 5.2 Endereçamento das interfaces dos roteadores

#### Router-RJ

| Interface | Endereço IP | Máscara | Função |
|---|---|---|---|
| `s0/0` | 172.16.100.1 | 255.255.255.0 | WAN para São Paulo |
| `f0/0` | 172.16.10.254 | 255.255.255.0 | LAN RJ-10 |
| `f0/1` | 172.16.20.254 | 255.255.255.0 | LAN RJ-20 |

#### Router-SP

| Interface | Endereço IP | Máscara | Função |
|---|---|---|---|
| `s0/0` | 172.16.100.2 | 255.255.255.0 | WAN para Rio de Janeiro |
| `f0/0` | 172.16.30.254 | 255.255.255.0 | LAN SP-30 |
| `f0/1` | 172.16.40.254 | 255.255.255.0 | LAN SP-40 |
| `s0/1` | 172.16.200.1 | 255.255.255.0 | WAN para Belo Horizonte |

#### Router-BH

| Interface | Endereço IP | Máscara | Função |
|---|---|---|---|
| `s0/0` | 172.16.200.2 | 255.255.255.0 | WAN para São Paulo |
| `f0/0` | 172.16.50.254 | 255.255.255.0 | LAN BH-50 |
| `f0/1` | 172.16.60.254 | 255.255.255.0 | LAN BH-60 |

### 5.3 Endereçamento dos hosts

| Host | Endereço IP | Máscara | Gateway |
|---|---|---|---|
| PC-RJ-10-1 | 172.16.10.1 | 255.255.255.0 | 172.16.10.254 |
| PC-RJ-10-2 | 172.16.10.2 | 255.255.255.0 | 172.16.10.254 |
| PC-RJ-20-1 | 172.16.20.1 | 255.255.255.0 | 172.16.20.254 |
| PC-RJ-20-2 | 172.16.20.2 | 255.255.255.0 | 172.16.20.254 |
| PC-SP-30-1 | 172.16.30.1 | 255.255.255.0 | 172.16.30.254 |
| PC-SP-30-2 | 172.16.30.2 | 255.255.255.0 | 172.16.30.254 |
| PC-SP-40-1 | 172.16.40.1 | 255.255.255.0 | 172.16.40.254 |
| PC-SP-40-2 | 172.16.40.2 | 255.255.255.0 | 172.16.40.254 |
| PC-BH-50-1 | 172.16.50.1 | 255.255.255.0 | 172.16.50.254 |
| PC-BH-50-2 | 172.16.50.2 | 255.255.255.0 | 172.16.50.254 |
| PC-BH-60-1 | 172.16.60.1 | 255.255.255.0 | 172.16.60.254 |
| PC-BH-60-2 | 172.16.60.2 | 255.255.255.0 | 172.16.60.254 |

---

## 6. Montagem no PNetLab

Montar a topologia com:

- 3 roteadores Cisco;
- 6 switches;
- 12 PCs;
- 2 enlaces WAN entre os roteadores;
- enlaces Ethernet entre roteadores, switches e hosts.

---

## 7. Configuração básica das interfaces

### 7.1 Router-RJ

```bash
Router> enable
Router# configure terminal
Router(config)# hostname Router-RJ
Router-RJ(config)# interface s0/0
Router-RJ(config-if)# ip address 172.16.100.1 255.255.255.0
Router-RJ(config-if)# no shut
Router-RJ(config-if)# interface f0/0
Router-RJ(config-if)# ip address 172.16.10.254 255.255.255.0
Router-RJ(config-if)# no shut
Router-RJ(config-if)# interface f0/1
Router-RJ(config-if)# ip address 172.16.20.254 255.255.255.0
Router-RJ(config-if)# no shut
Router-RJ(config-if)# end

```

### 7.2 Router-SP

```bash
Router> enable
Router# configure terminal
Router(config)# hostname Router-SP
Router-SP(config)# interface s0/0
Router-SP(config-if)# ip address 172.16.100.2 255.255.255.0
Router-SP(config-if)# clock rate 500000
Router-SP(config-if)# no shut
Router-SP(config-if)# interface s0/1
Router-SP(config-if)# ip address 172.16.200.1 255.255.255.0
Router-SP(config-if)# clock rate 500000
Router-SP(config-if)# no shut
Router-SP(config-if)# interface f0/0
Router-SP(config-if)# ip address 172.16.30.254 255.255.255.0
Router-SP(config-if)# no shut
Router-SP(config-if)# interface f0/1
Router-SP(config-if)# ip address 172.16.40.254 255.255.255.0
Router-SP(config-if)# no shut
Router-SP(config-if)# end

```

### 7.3 Router-BH

```bash
Router> enable
Router# configure terminal
Router(config)# hostname Router-BH
Router-BH(config)# interface s0/0
Router-BH(config-if)# ip address 172.16.200.2 255.255.255.0
Router-BH(config-if)# no shut
Router-BH(config-if)# interface f0/0
Router-BH(config-if)# ip address 172.16.50.254 255.255.255.0
Router-BH(config-if)# no shut
Router-BH(config-if)# interface f0/1
Router-BH(config-if)# ip address 172.16.60.254 255.255.255.0
Router-BH(config-if)# no shut
Router-BH(config-if)# end

```

---

## 8. Verificação inicial sem roteamento dinâmico

Antes de configurar RIP ou OSPF, verificar:

- conectividade apenas entre redes diretamente conectadas;
- ausência de rotas remotas;
- tabela de rotas contendo apenas rotas conectadas.

### Comando de verificação

```bash
show ip route
```

### Resultado esperado

- presença de rotas `C` e `L`;
- ausência de rotas aprendidas dinamicamente;
- pings bem-sucedidos apenas para redes diretamente conectadas.

---

## 9. Configuração do RIP

Como o cenário usa a faixa `172.16.0.0/16`, pode-se anunciar a rede maior `172.16.0.0` no processo RIP.

### 9.1 Router-RJ

```bash
Router-RJ> enable
Router-RJ# configure terminal
Router-RJ(config)# router rip
Router-RJ(config-router)# network 172.16.0.0
Router-RJ(config-router)# end
Router-RJ#

```

### 9.2 Router-SP

```bash
Router-SP> enable
Router-SP# configure terminal
Router-SP(config)# router rip
Router-SP(config-router)# network 172.16.0.0
Router-SP(config-router)# end
Router-SP#
```

### 9.3 Router-BH

```bash
Router-BH> enable
Router-BH# configure terminal
Router-BH(config)# router rip
Router-BH(config-router)# network 172.16.0.0
Router-BH(config-router)# end
Router-BH#
```

---

## 10. Verificação do RIP

### Comandos

```bash
show ip route
show ip protocols
ping 172.16.30.1
ping 172.16.40.1
ping 172.16.50.1
ping 172.16.60.1
```

### Resultado esperado

- rotas aprendidas com marcação `R`;
- comunicação fim a fim entre as redes;
- percepção de que o RIP usa contagem de saltos como métrica.

---

## 11. Remoção do RIP

Em cada roteador:

```bash
configure terminal
no router rip
end
write memory
```

---

## 12. Configuração do OSPF

Neste laboratório será utilizada apenas a **área 0**, com objetivo introdutório.

### 12.1 Router-RJ

```bash
Router-RJ> enable
Router-RJ# configure terminal
Router-RJ(config)# router ospf 64
Router-RJ(config-router)# network 172.16.10.0 0.0.0.255 area 0
Router-RJ(config-router)# network 172.16.20.0 0.0.0.255 area 0
Router-RJ(config-router)# network 172.16.100.0 0.0.0.255 area 0
Router-RJ(config-router)# end
```

### 12.2 Router-SP

```bash
Router-SP> enable
Router-SP# configure terminal
Router-SP(config)# router ospf 65
Router-SP(config-router)# network 172.16.30.0 0.0.0.255 area 0
Router-SP(config-router)# network 172.16.40.0 0.0.0.255 area 0
Router-SP(config-router)# network 172.16.100.0 0.0.0.255 area 0
Router-SP(config-router)# network 172.16.200.0 0.0.0.255 area 0
Router-SP(config-router)# end
```

### 12.3 Router-BH

```bash
Router-BH> enable
Router-BH# configure terminal
Router-BH(config)# router ospf 66
Router-BH(config-router)# network 172.16.50.0 0.0.0.255 area 0
Router-BH(config-router)# network 172.16.60.0 0.0.0.255 area 0
Router-BH(config-router)# network 172.16.200.0 0.0.0.255 area 0
Router-BH(config-router)# end

```

---

## 13. Verificação do OSPF

### Comandos em todos os roteadores

```bash
show ip ospf neighbor
show ip route
show ip protocols
ping 172.16.30.1
ping 172.16.40.1
ping 172.16.50.1
ping 172.16.60.1
```

### Resultado esperado

- adjacência OSPF formada entre RJ-SP e SP-BH;
- rotas aprendidas com marcação `O`;
- conectividade total entre todas as LANs.

---

## 14. Testes sugeridos

### 14.1 Teste de alcance entre unidades

A partir de um host do Rio de Janeiro, testar conectividade com São Paulo e Belo Horizonte.

Exemplo a partir do PC-RJ-10-1:

```bash
ping 172.16.30.1
ping 172.16.40.1
ping 172.16.50.1
ping 172.16.60.1
```

### 14.2 Teste de tabela de rotas

```bash
show ip route
```

### 14.3 Teste de vizinhança OSPF

```bash
show ip ospf neighbor
```

---

## 15. Comparação orientada entre RIP e OSPF

Após concluir as duas configurações, responder:

1. Qual protocolo foi mais simples de configurar?
2. Qual protocolo apresentou maior riqueza de informações operacionais?
3. Qual a principal métrica do RIP?
4. Qual algoritmo é usado pelo OSPF?
5. Qual protocolo tende a escalar melhor?
6. Qual protocolo converge melhor em cenários maiores?

---

## 16. Troubleshooting proposto

### Situação 1 – Remover uma rede do anúncio OSPF em São Paulo

```bash
configure terminal
router ospf 1
 no network 172.16.40.0 0.0.0.255 area 0
end
```

Verificar:

- perda de rota para a rede `172.16.40.0/24`;
- impacto na conectividade;
- necessidade de restaurar a configuração.

### Situação 2 – Interromper o enlace SP-BH

```bash
configure terminal
interface s0/1
 shutdown
end
```

Verificar:

- perda da adjacência com BH;
- remoção de rotas remotas;
- impacto nos testes de ping.

Restaurar:

```bash
configure terminal
interface s0/1
 no shutdown
end
```

### Situação 3 – Remover o RIP de um roteador

```bash
configure terminal
no router rip
end
```

Verificar o impacto na propagação de rotas.

---

## 17. Critérios de avaliação

| Critério | Pontos |
|---|---:|
| Configuração correta das interfaces | 2,0 |
| Funcionamento do RIP | 2,0 |
| Funcionamento do OSPF | 2,0 |
| Testes de conectividade | 2,0 |
| Análise comparativa RIP x OSPF | 2,0 |

**Total: 10,0**

---

## 18. Entregáveis

- print da topologia no PNetLab;
- print do `show ip route` com RIP;
- print do `show ip route` com OSPF;
- print do `show ip ospf neighbor`;
- breve relatório contendo:
  - objetivo;
  - topologia;
  - comandos aplicados;
  - testes realizados;
  - comparação entre RIP e OSPF.

---

## 19. Referências

- BRITO, Samuel Henrique Bucke. *Laboratórios de Tecnologias Cisco em Infraestrutura de Redes*. 2. ed. São Paulo: Novatec, 2014.
- LOBATO, Luiz Carlos. *Protocolos de Roteamento IP*. Rio de Janeiro: RNP/ESR, 2013.
- KUROSE, James F.; ROSS, Keith W. *Redes de Computadores e a Internet: uma abordagem top-down*. 8. ed. Porto Alegre: Pearson/Bookman, 2021.
