# Laboratório 01 — Configuração básica de roteadores no PNetLab

**Disciplina:** ENE0025 - Protocolos de Transporte e Roteamento  
**Professor responsável:** **Prof. Dr. Laerte Peotta de Melo**

---

## 1. Visão geral

Este laboratório tem como objetivo introduzir o aluno à configuração inicial de roteadores Cisco em ambiente emulado no **PNetLab**. A prática cobre acesso à CLI, definição de parâmetros básicos de administração, configuração de interface IP, habilitação de acesso remoto seguro e testes de conectividade.

Este é o primeiro cenário prático da disciplina e serve como base para os laboratórios posteriores de roteamento estático e dinâmico.

---

## 2. Objetivos

Ao final deste laboratório, o aluno deverá ser capaz de:

- acessar o roteador pela CLI;
- configurar o nome do equipamento;
- proteger o acesso local e privilegiado;
- configurar banner de aviso;
- atribuir endereço IP a uma interface;
- ativar a interface de rede;
- habilitar acesso remoto via SSH;
- validar conectividade com um host da LAN;
- salvar a configuração no equipamento.

---

## 3. Topologia

A topologia do laboratório é composta por:

- **1 roteador Cisco**
- **1 switch Ethernet**
- **1 host VPCS**
- enlaces Ethernet entre os dispositivos

### Diagrama lógico

```text
+--------+        +----------+        +--------+
|  PC1   |------->|  Switch  |<-------|   R1   |
| VPCS   |        |   SW1    |        | G0/0   |
+--------+        +----------+        +--------+
```

---

## 4. Endereçamento IP

| Dispositivo | Interface | Endereço IP   | Máscara       | Gateway       |
|-------------|-----------|---------------|---------------|---------------|
| R1          | G0/0      | 192.168.0.254 | 255.255.255.0 | —             |
| PC1         | eth0      | 192.168.0.10  | 255.255.255.0 | 192.168.0.254 |

---

## 5. Requisitos

- Ambiente **PNetLab** operacional
- Imagem de roteador Cisco compatível:
  - IOSv, CSR1000v, IOL ou equivalente
- 1 nó **Ethernet Switch**
- 1 nó **VPCS**
- Acesso ao console do roteador pelo navegador

---

## 6. Montagem do cenário no PNetLab

1. Criar um novo laboratório no PNetLab.
2. Adicionar os seguintes nós:
   - 1 roteador Cisco
   - 1 switch
   - 1 VPCS
3. Conectar:
   - `PC1` ao `SW1`
   - `R1 G0/0` ao `SW1`
4. Inicializar os dispositivos.

---

## 7. Configuração do roteador

Acessar o console do roteador e executar os comandos abaixo.

### 7.1 Configuração inicial

```bash
enable
configure terminal
hostname R1
no ip domain-lookup
banner motd #
Acesso restrito. Somente usuarios autorizados.
#
enable secret unb123
service password-encryption
```

### 7.2 Configuração da linha de console

```bash
line console 0
password cisco
login
logging synchronous
exec-timeout 10 0
exit
```

### 7.3 Criação de usuário local e habilitação de SSH

```bash
username admin privilege 15 secret Admin@123
ip domain-name unb.lab
crypto key generate rsa
1024
ip ssh version 2
line vty 0 4
login local
transport input ssh
exec-timeout 10 0
logging synchronous
exit
```

### 7.4 Configuração da interface LAN

> Caso a interface disponível no seu roteador tenha outro nome, como `f0/0`, adapte os comandos.

```bash
interface g0/0
description LAN-PNETLAB
ip address 192.168.0.254 255.255.255.0
no shutdown
exit
end
copy running-config startup-config
```

---

## 8. Configuração do host VPCS

No terminal do **PC1**, executar:

```bash
ip 192.168.0.10/24 192.168.0.254
save
```

---

## 9. Comandos de verificação

### 9.1 No roteador

```bash
show ip interface brief
show running-config
show users
show ssh
```

### 9.2 No PC1

```bash
ping 192.168.0.254
```

Caso o ambiente permita cliente SSH no host utilizado, testar também:

```bash
ssh -l admin 192.168.0.254
```

---

## 10. Resultados esperados

Ao concluir a prática, o aluno deve observar que:

- a interface `G0/0` do roteador está em estado `up/up`;
- o host `PC1` consegue alcançar o endereço `192.168.0.254`;
- o roteador aceita conexões SSH;
- a configuração foi salva corretamente.

---

## 11. Checklist de validação

Marque cada item após a execução:

- [ ] Topologia criada corretamente no PNetLab  
- [ ] Roteador nomeado como `R1`  
- [ ] Senha privilegiada configurada  
- [ ] Usuário local `admin` criado  
- [ ] Banner de aviso configurado  
- [ ] Interface LAN configurada com IP `192.168.0.254/24`  
- [ ] Interface ativada com `no shutdown`  
- [ ] SSH habilitado  
- [ ] Host configurado com IP `192.168.0.10/24`  
- [ ] Ping realizado com sucesso  
- [ ] Configuração salva com sucesso  

---

## 12. Questões para reflexão

1. Qual a diferença entre acesso **via console** e acesso **remoto pela rede**?
2. Qual a função do comando `no ip domain-lookup` em laboratório?
3. Por que o comando `enable secret` é preferível ao `enable password`?
4. Por que o protocolo SSH é mais seguro que o Telnet?
5. O que ocorre se a interface não receber o comando `no shutdown`?

---

## 13. Entrega

O aluno deverá entregar:

- captura de tela da topologia no PNetLab;
- saída do comando `show ip interface brief`;
- saída do comando `show running-config`;
- evidência do teste de `ping`;
- evidência do acesso remoto via SSH, quando aplicável.

---

## 14. Desafio extra

Realize as modificações abaixo:

1. Altere o hostname do roteador para `R1-LAB`.
2. Configure uma descrição mais detalhada na interface.
3. Mude o endereço IP da interface do roteador para `192.168.10.254/24`.
4. Ajuste o IP do PC para a nova rede.
5. Repita todos os testes de conectividade.

---

## 15. Conclusão

Este laboratório introduz a configuração básica de roteadores no PNetLab e estabelece as competências mínimas necessárias para os próximos cenários da disciplina. Dominar CLI, interface IP, acesso remoto e testes de conectividade é essencial antes da evolução para protocolos de roteamento e atividades de diagnóstico mais avançadas.
