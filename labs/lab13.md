# Laboratório 13 - Testes de Ataques Web e Análise de Logs no WAF

**Disciplina:** ENE0025 - Protocolos de Transporte e Roteamento  
**Professor responsável:** Prof. Dr. Laerte Peotta de Melo  
**Tema:** Validação prática de proteção web com WAF  

---

## 1. Objetivo

Dar continuidade ao **Lab 12**, realizando testes controlados de requisições HTTP legítimas e maliciosas contra o ambiente protegido por **Nginx + ModSecurity + OWASP CRS**, de modo que o estudante consiga:

- observar o comportamento do WAF diante de tráfego normal e suspeito;
- identificar bloqueios gerados por regras do OWASP CRS;
- analisar logs do Nginx e do ModSecurity;
- compreender, na prática, a diferença entre:
  - firewall de pacotes;
  - firewall stateful;
  - WAF.

---

## 2. Introdução

No **Lab 12**, o aluno implantou um **proxy reverso com WAF**, posicionando o Linux WAF entre o cliente e o servidor web interno. Nessa etapa, o foco foi a instalação, a publicação do serviço e a validação do acesso legítimo.

No **Lab 13**, o foco passa a ser o **comportamento do WAF diante de diferentes tipos de requisição HTTP**. O objetivo é mostrar que, mesmo com a porta 80 liberada e a comunicação TCP funcionando normalmente, ainda assim uma requisição pode ser bloqueada quando apresenta sinais de ataque na camada de aplicação.

Este laboratório é importante porque torna visível a principal diferença entre os mecanismos estudados até aqui:

- o firewall de pacotes decide com base em IP, protocolo e porta;
- o firewall stateful acompanha o estado da conexão;
- o WAF inspeciona o conteúdo da requisição HTTP.

---

## 3. Relação com os Labs 10, 11 e 12

| Laboratório | Tecnologia | Foco principal |
|---|---|---|
| Lab 10 | Firewall de pacotes | Filtragem por IP, protocolo e porta |
| Lab 11 | Firewall stateful | Filtragem com rastreamento de conexão |
| Lab 12 | WAF em proxy reverso | Publicação e inspeção HTTP |
| Lab 13 | WAF com testes práticos | Bloqueio e análise de requisições maliciosas |

---

## 4. Situação-problema

Uma aplicação web já está publicada por meio de um proxy reverso com WAF. A equipe de segurança precisa verificar se essa proteção consegue identificar padrões suspeitos em requisições HTTP e registrar adequadamente os eventos para investigação.

Para isso, serão realizados acessos normais e testes controlados com padrões frequentemente associados a:

- **SQL Injection:** Ataque em que o invasor tenta inserir comandos SQL em campos ou parâmetros da aplicação para manipular consultas ao banco de dados.
- **Cross-Site Scripting (XSS):** Ataque em que código malicioso, geralmente JavaScript, é injetado em páginas web para ser executado no navegador da vítima.
- **path traversal:** Ataque em que o invasor tenta acessar arquivos ou diretórios indevidos no servidor usando caminhos como `../` para sair da pasta permitida.
- **Parâmetros suspeitos em URL:** São valores passados na própria URL que indicam tentativa de manipular o comportamento da aplicação ou acessar algo indevido.

### 4.1 Exemplos comuns

- tentar inserir comandos em parâmetros, como em `?id=1' OR '1'='1`
- enviar scripts maliciosos, como em `?q=<script>alert(1)</script>`
- tentar acessar arquivos fora da pasta permitida, como em `?file=../../etc/passwd`

São chamados de suspeitos porque fogem do uso normal esperado pela aplicação e podem indicar tentativa de ataque, exploração ou varredura. Em um **WAF**, esses parâmetros são analisados para decidir se a requisição deve ser permitida, registrada ou bloqueada.

---

## 5. Topologia lógica

```mermaid
flowchart LR
    CLIENT["Linux Cliente<br/>192.168.10.10/24"]
    WAF["Linux WAF<br/>eth0: 192.168.10.1/24<br/>eth1: 192.168.20.1/24<br/>Nginx + ModSecurity + CRS"]
    WEB["Servidor Web Interno<br/>192.168.20.10/24"]

    CLIENT --- WAF
    WAF --- WEB

    classDef host fill:#fef3c7,stroke:#d97706,color:#111827,stroke-width:1.5px;
    classDef waf fill:#dbeafe,stroke:#1d4ed8,color:#111827,stroke-width:2px;
    classDef web fill:#dcfce7,stroke:#16a34a,color:#111827,stroke-width:1.5px;

    class CLIENT host;
    class WAF waf;
    class WEB web;
```

---

## 6. Plano de endereçamento

| Dispositivo | Interface | Endereço IP | Máscara | Gateway |
|---|---|---|---|---|
| Linux Cliente | eth0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| Linux WAF | eth0 | 192.168.10.1 | 255.255.255.0 | - |
| Linux WAF | eth1 | 192.168.20.1 | 255.255.255.0 | - |
| Servidor Web Interno | eth0 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |

---

## 7. Pré-requisitos

Este laboratório pressupõe que o **Lab 12** já esteja funcional, com os seguintes itens concluídos:

- endereçamento IP configurado;
- servidor web interno respondendo;
- Nginx em funcionamento no host WAF;
- ModSecurity habilitado;
- OWASP CRS habilitado;
- proxy reverso funcionando;
- acesso normal ao serviço via `http://192.168.10.1`.

---

## 8. Validação inicial do ambiente

Antes dos testes de ataque, confirme que o ambiente está operacional.

### 8.1 No servidor web interno

```bash
curl http://127.0.0.1
```

### 8.2 No cliente, via WAF

```bash
curl http://192.168.10.1
```

Resultado esperado: a página do servidor interno deve ser entregue normalmente.

---

## 9. Verificação dos serviços no WAF

```bash
sudo systemctl status nginx
sudo nginx -t
```

Verificar se o ModSecurity está ativo no Nginx:

```bash
sudo grep -R "modsecurity on" /etc/nginx/
```

Verificar o arquivo principal de regras:

```bash
sudo cat /etc/nginx/modsec/main.conf
```

---

## 10. Observação de logs

Durante os testes, mantenha ao menos um terminal aberto no WAF para acompanhar os registros.

### 10.1 Log de acesso do Nginx

```bash
sudo tail -f /var/log/nginx/access.log
```

### 10.2 Log de erro do Nginx

```bash
sudo tail -f /var/log/nginx/error.log
```

### 10.3 Log do ModSecurity

Dependendo da configuração:

```bash
sudo tail -f /var/log/modsec_audit.log
```

ou verifique no `modsecurity.conf` o caminho configurado para auditoria.

---

## 11. Testes de tráfego legítimo

### 11.1 Acesso normal à página inicial

No cliente:

```bash
curl http://192.168.10.1
```

### 11.2 Acesso com parâmetro simples

```bash
curl "http://192.168.10.1/?id=10"
```

### 11.3 Acesso com user-agent comum

```bash
curl -A "Mozilla/5.0" http://192.168.10.1
```

### 11.4 Resultado esperado

- a resposta deve ser permitida;
- o acesso deve aparecer nos logs;
- não deve haver bloqueio para requisições normais.

---

## 12. Testes controlados de requisições suspeitas

> Execute os testes abaixo somente no ambiente de laboratório.

### 12.1 Teste de padrão semelhante a SQL Injection

```bash
curl "http://192.168.10.1/?id=1' OR '1'='1"
```

### 12.2 Teste de padrão semelhante a XSS

```bash
curl "http://192.168.10.1/?q=<script>alert(1)</script>"
```

### 12.3 Teste de path traversal

```bash
curl "http://192.168.10.1/../../../etc/passwd"
```

### 12.4 Teste com URL codificada

```bash
curl "http://192.168.10.1/?file=%2e%2e%2f%2e%2e%2fetc%2fpasswd"
```

### 12.5 Resultado esperado

Em parte dos casos, o WAF deverá:

- bloquear a requisição;
- registrar o evento no log;
- associar o evento a alguma regra do CRS;
- retornar resposta como `403 Forbidden` ou comportamento equivalente.

> Alguns testes podem não acionar bloqueio dependendo da versão do CRS, do modo configurado e do conjunto de regras habilitado. Isso também deve ser analisado pelos alunos.

---

## 13. Comparação prática: rede funcionando, aplicação bloqueada


### 13.1 Verificar que a conectividade IP continua válida

No cliente:

```bash
ping 192.168.10.1
```

### 13.2 Verificar que a porta HTTP continua publicada

```bash
curl -I http://192.168.10.1
```

### 13.3 Verificar que uma requisição maliciosa pode ser bloqueada mesmo com a porta aberta

```bash
curl "http://192.168.10.1/?q=<script>alert(1)</script>"
```

### 13.4 Interpretação

O aluno deve perceber que:

- a rede está operacional;
- a sessão TCP pode ser estabelecida;
- a porta 80 está acessível;
- mesmo assim, a requisição pode ser barrada por causa do conteúdo HTTP.

---

## 14. Atividade comparativa entre os modelos de firewall

Preencha a tabela abaixo após executar os testes.

| Situação | Firewall de pacotes | Firewall stateful | WAF |
|---|---|---|---|
| Liberar ou bloquear TCP/80 | Sim | Sim | Sim |
| Acompanhar conexão já estabelecida | Não | Sim | Indiretamente |
| Analisar conteúdo da URL | Não | Não | Sim |
| Identificar padrão de XSS | Não | Não | Sim |
| Identificar padrão de SQLi | Não | Não | Sim |
| Bloquear requisição web suspeita sem fechar a porta 80 | Não | Não | Sim |

---

## 15. Análise de logs

Após os testes, os alunos devem localizar nos logs:

- horário da requisição;
- IP de origem;
- recurso acessado;
- código HTTP retornado;
- regra acionada, quando disponível;
- mensagem de auditoria do ModSecurity.

### 15.1 Exemplo de análise esperada

O aluno deve conseguir afirmar algo como:

- uma requisição legítima foi aceita e registrada no `access.log`;
- uma requisição suspeita gerou evento no `modsec_audit.log`;
- o WAF atuou antes do servidor processar normalmente o conteúdo.

---

## 16. Atividade opcional de tuning

Caso o ambiente esteja estável, o professor pode pedir uma pequena atividade de ajuste:

- identificar uma regra que gerou bloqueio;
- analisar a mensagem registrada;
- discutir se o evento representa ataque real, falso positivo ou teste didático;
- propor ajuste futuro sem desabilitar a proteção inteira.

> Esta etapa é opcional.
> 
---

## 17. Questões para análise

1. O que diferencia um WAF de um firewall stateful?
2. Por que a requisição pode ser bloqueada mesmo com a porta 80 aberta?
3. O que o WAF consegue observar que o `iptables` não observa?
4. Qual foi o comportamento do ambiente diante de tráfego legítimo?
5. Qual foi o comportamento do ambiente diante de tráfego suspeito?
6. O que os logs do Nginx mostram?
7. O que os logs do ModSecurity mostram?
8. Um bloqueio HTTP significa necessariamente falha da rede? Explique.
9. O WAF substitui a necessidade de correção de vulnerabilidade na aplicação? Explique.
10. Qual foi a principal evidência prática de que o WAF atua na camada de aplicação?

---

## 18. Critérios de avaliação

| Critério | Pontuação |
|---|---:|
| Validação e operacionalidade do ambiente herdado do Lab 12 | 1,5 |
| Execução correta dos testes de tráfego legítimo (curl / HTTP) | 1,5 |
| Execução dos testes de tráfego malicioso (SQLi, XSS, Path Traversal) | 2,5 |
| Coleta, análise e correlação dos logs do Nginx e ModSecurity | 2,5 |
| Respostas técnicas às questões para análise e tabela comparativa | 2,0 |
| **Total** | **10,0** |

---

## 19. Entregáveis

Cada aluno deve entregar:

- print da topologia no PNetLab;
- print do acesso normal ao serviço via WAF;
- print de pelo menos um teste suspeito executado;
- print dos logs do Nginx;
- print dos logs do ModSecurity;
- tabela comparativa preenchida;
- relatório curto explicando:
  - o que foi permitido;
  - o que foi bloqueado;
  - por que isso demonstra a atuação do WAF;
- respostas completas às questões para análise.

---

## 20. Conclusão esperada

Ao final deste laboratório, o estudante deve concluir que:

- o WAF atua em um nível diferente dos firewalls estudados anteriormente;
- a proteção web depende da inspeção do conteúdo HTTP;
- é possível manter um serviço publicado e, ao mesmo tempo, bloquear padrões suspeitos;
- logs são parte essencial da operação e da investigação;
- o **Lab 13** torna visível, de forma prática, a diferença entre **permitir a conexão** e **permitir a requisição**.

---

[← Anterior: Laboratório 12](lab12.md) | [Índice Geral (README)](../README.md) | [Próximo: Laboratório 14 →](lab14.md)

