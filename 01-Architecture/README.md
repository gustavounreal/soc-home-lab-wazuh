# SOC Home Lab — Arquitetura e Infraestrutura

![Status](https://img.shields.io/badge/Status-Em_Desenvolvimento-yellow)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-blue)
![VirtualBox](https://img.shields.io/badge/Hypervisor-VirtualBox_7.x-orange)
![Windows](https://img.shields.io/badge/OS-Windows_11_Pro-0078D6?logo=windows)
![Linux](https://img.shields.io/badge/OS-Ubuntu_24.04_LTS-E95420?logo=ubuntu)
![Kali](https://img.shields.io/badge/OS-Kali_Linux-557C94?logo=kali-linux)

**Projeto:** SOC Home Lab — Security Monitoring & Threat Detection  
**Categoria:** Cybersecurity | Security Operations Center (SOC)  
**Ambiente:** Virtualizado e isolado (4 Virtual Machines)

---

## 1. Visão Geral

Este projeto consiste na construção de um laboratório prático de Security Operations Center (SOC), desenvolvido para simular atividades de monitoramento de segurança, coleta de eventos, análise de logs e detecção de ameaças em um ambiente controlado.

A infraestrutura utiliza o **Wazuh** como plataforma SIEM, centralizando os eventos gerados por diferentes sistemas operacionais e permitindo o monitoramento contínuo dos endpoints.

O ambiente foi estruturado para reproduzir componentes e fluxos de trabalho encontrados em operações de segurança corporativas, proporcionando experiência prática com monitoramento, identificação de atividades suspeitas e análise de eventos.

**Principais objetivos:**
- Implementar uma infraestrutura centralizada de monitoramento de segurança.
- Integrar endpoints Windows e Linux ao SIEM.
- Coletar e analisar eventos de segurança.
- Identificar atividades suspeitas por meio de regras e alertas.
- Simular cenários controlados de incidentes.
- Investigar eventos e interpretar alertas de segurança.
- Desenvolver habilidades práticas relacionadas à rotina de um SOC Analyst.

### Visão Geral da Arquitetura

![Imagem 01 — SOC Architecture](<./imagens/Imagem 01 — SOC Architecture.png>)

*Figura 01 — Visão geral da arquitetura do laboratório SOC.*

---

## 2. Arquitetura

A arquitetura foi organizada em três camadas principais, cada uma com responsabilidades específicas dentro do ambiente.

### 2.1 Camada SIEM e Monitoramento
Camada responsável pela centralização, processamento, correlação e visualização dos eventos de segurança.

**Componente principal:** Wazuh Server.

**Responsabilidades:**
- Recebimento de eventos dos endpoints.
- Gerenciamento dos agentes Wazuh.
- Análise e classificação de eventos.
- Aplicação de regras de detecção.
- Geração e armazenamento de alertas.
- Disponibilização de dashboards de monitoramento.

### 2.2 Camada de Monitoramento dos Endpoints
Camada composta pelos sistemas operacionais monitorados, responsáveis pela geração e envio dos eventos de segurança.

**Componentes:** Windows Endpoint e Ubuntu Linux Endpoint.

**Responsabilidades:**
- Geração de eventos operacionais e de segurança.
- Registro de atividades de autenticação.
- Monitoramento de alterações no sistema.
- Coleta de logs.
- Envio de eventos ao servidor central.

### 2.3 Camada de Testes de Segurança
Camada destinada à execução controlada de atividades de reconhecimento, testes de segurança e geração de eventos para validação dos mecanismos de monitoramento.

**Componente:** Kali Linux.

**Responsabilidades:**
- Reconhecimento controlado do ambiente.
- Geração de atividades para testes.
- Simulação de tentativas de autenticação.
- Execução de testes de segurança autorizados.
- Validação da capacidade de observação e detecção do SIEM.

### Arquitetura Lógica

![Imagem 02 — Network Topology](<./imagens/Imagem 02 — Network Topology.png>)

*Figura 02 — Topologia lógica e comunicação entre os componentes.*

---

## 3. Componentes

A infraestrutura é composta por quatro máquinas virtuais, distribuídas em duas redes distintas para separar o tráfego de monitoramento da interface de gerenciamento:

| Component | Operating System | IP (Internal Network) | Role |
|---|---|---|---|
| **SOC-SERVER** | Ubuntu Server 24.04 LTS | `10.10.10.10` | SIEM, gerenciamento centralizado e Dashboard |
| **WIN-ENDPOINT** | Windows 11 Pro | `10.10.10.20` | Endpoint monitorado (Windows) |
| **LINUX-ENDPOINT** | Ubuntu Server 24.04 LTS | `10.10.10.30` | Endpoint monitorado (Linux) |
| **KALI-ATTACKER** | Kali Linux | `10.10.10.40` | Estação de testes controlados de segurança |

### 3.1 Servidor Wazuh
Servidor central responsável pela infraestrutura de monitoramento e análise de eventos de segurança.

**Principais funções:**
- Recebimento de eventos dos agentes.
- Gerenciamento e monitoramento dos endpoints.
- Processamento e análise de eventos.
- Aplicação de regras de detecção.
- Geração de alertas de segurança.
- Monitoramento centralizado por meio de dashboards.
- Análise de vulnerabilidades dos endpoints, conforme os recursos habilitados.

**Componentes da plataforma:** Wazuh Manager, Wazuh Indexer e Wazuh Dashboard.

![Imagem 03 — Wazuh Server](<./imagens/Imagem 03 — Wazuh Server.png>)

*Figura 03 — Captura histórica do Dashboard Wazuh. A imagem mostra a interface acessível, mas registra zero agentes e zero eventos; não é usada como comprovação da operação atual.*

### 3.2 Endpoint Windows
Sistema Windows integrado ao ambiente de monitoramento por meio do Wazuh Agent.

**Fontes de eventos monitoradas:**
- Windows Event Logs.
- Eventos de autenticação.
- Tentativas de acesso.
- Alterações de usuários e grupos.
- Atividades relacionadas à segurança do sistema.
- Eventos de serviços e processos.

### 3.3 Endpoint Ubuntu Linux
Sistema Linux integrado ao monitoramento centralizado por meio do Wazuh Agent.

**Fontes de eventos monitoradas:**
- Logs de autenticação.
- Registros SSH.
- Atividades de usuários.
- Eventos de serviços.
- Alterações de arquivos e permissões.
- Eventos relacionados à administração do sistema.

### 3.4 Kali Linux
Sistema operacional utilizado como estação de testes de segurança.

**Finalidades:**
- Reconhecimento controlado do ambiente.
- Geração de atividades para testes de monitoramento.
- Simulação de tentativas de autenticação.
- Execução de testes de segurança autorizados.
- Validação da geração e identificação de eventos no SIEM.

*O Kali Linux atua como estação de testes. Na execução registrada nas evidências, o `KALI-ATTACKER` também aparece como agente Wazuh ativo; essa condição depende da configuração efetivamente instalada no ambiente.*

---

## 4. Infraestrutura

O laboratório utiliza virtualização para executar os diferentes sistemas operacionais em um ambiente controlado, permitindo a integração dos componentes e a simulação de cenários de monitoramento de segurança.

### 4.1 Ambiente de Virtualização

| Resource | Configuration |
|---|---|
| Hypervisor | Type 2 Hypervisor |
| Host Operating System | Windows 11 Pro 64-bit |
| Virtualization Platform | Oracle VM VirtualBox 7.x |
| Network Mode | Internal Network + NAT + Host-Only |
| Host RAM | 32 GB DDR4/DDR5 |
| Host Storage | 512 GB SSD |
| Virtual Machines | 4 |

### 4.2 Recursos das Máquinas Virtuais

A distribuição de recursos foi dimensionada para um computador doméstico com 32 GB de RAM, permitindo a execução simultânea das máquinas virtuais e do sistema operacional hospedeiro.

| Virtual Machine | vCPU | RAM | Storage |
|---|---:|---:|---:|
| SOC-SERVER | 4 | 8 GB | 80 GB |
| WIN-ENDPOINT | 2 | 6 GB | 64 GB |
| LINUX-ENDPOINT | 2 | 2 GB | 25 GB |
| KALI-ATTACKER | 2 | 3 GB | 30 GB |
| **Total** | **10** | **19 GB** | **199 GB** |

**Resource Allocation:**
- 19 GB de RAM alocados às máquinas virtuais.
- Aproximadamente 13 GB de RAM reservados ao sistema operacional hospedeiro e demais aplicações.
- 10 vCPUs distribuídos entre as quatro máquinas virtuais.
- 199 GB de armazenamento virtual provisionado.

*O armazenamento virtual pode utilizar alocação dinâmica no VirtualBox, reduzindo o espaço físico efetivamente ocupado enquanto os discos não atingirem sua capacidade máxima.*

### 4.3 Ambiente do Laboratório

![Imagem 06 — Laboratório SOC com Quatro Máquinas Virtuais](<./imagens/Imagem 06 —Laboratório SOC com Quatro Máquinas Virtuais.png>)

*Figura 06 — Evidência consolidada da execução das quatro VMs, dos IPs e rotas observados, da conectividade interna, da comunicação TCP com o Manager na porta 1514 e do Dashboard em tema claro com três agentes ativos.*

---

## 5. Topologia de Rede

A comunicação entre os sistemas ocorre por meio de uma infraestrutura de rede virtual configurada no Oracle VM VirtualBox (`VirtualBox`). O ambiente utiliza uma rede interna dedicada para a comunicação entre os componentes e uma interface Host-Only para acesso administrativo seguro.

### 5.1 Configuração de Rede

- **Network Name:** SOC-LAB
- **Network Address:** 10.10.10.0/24
- **Subnet Mask:** 255.255.255.0

| Component | Hostname | IP Address (Internal) | IP Address (Host-Only) |
|---|---|---|---|
| Wazuh Server | SOC-SERVER | `10.10.10.10` | `192.168.56.10` |
| Windows Endpoint | WIN-ENDPOINT | `10.10.10.20` | — |
| Ubuntu Endpoint | LINUX-ENDPOINT | `10.10.10.30` | — |
| Kali Linux | KALI-ATTACKER | `10.10.10.40` | — |

### 5.2 Interfaces de Rede

| Interface | Configuration | Purpose |
|---|---|---|
| **Internal Network** | `SOC-LAB` (`10.10.10.0/24`) | Comunicação isolada e tráfego de monitoramento entre as VMs |
| **NAT** | VirtualBox NAT | Acesso externo para atualizações e instalação de pacotes nos nós |
| **Host-Only** | VirtualBox Host-Only (`192.168.56.0/24`) | Acesso seguro do host físico (Windows) ao Wazuh Dashboard (`192.168.56.10`) |

- O **SOC-SERVER** utiliza a interface Host-Only (`192.168.56.10`) para permitir o acesso ao Wazuh Dashboard diretamente pelo navegador do computador hospedeiro com total segurança.
- A interface `Internal Network` (`10.10.10.x`) gerencia o tráfego de agentes e testes entre o SOC, os endpoints e o Kali Linux.
- A interface NAT permite acesso externo quando necessário para atualização dos sistemas e instalação de dependências.

### 5.3 Evidências de Execução

#### Adaptadores de rede do VirtualBox

![Imagem 05 — Interfaces configuradas no VirtualBox](<./imagens/Imagem 05 — interfaces configuradas no VirtualBox.jpg>)

*Figura 05 — Configuração observada dos adaptadores no VirtualBox em 26/10/2026: `SOC-LAB` para a rede interna, NAT para acesso externo e Host-Only no `SOC-SERVER`. A imagem comprova a configuração dos adaptadores, não substitui os testes de conectividade.*

#### Validação de NAT e DNS

![Imagem 07 — Infográfico do Laboratório SOC-LAB Conectado](<./imagens/Imagem 07 —Infográfico do Laboratório SOC-LAB Conectado.png>)

*Figura 07 — Evidência composta de execução: os terminais exibem ping para `8.8.8.8` e resolução DNS de `google.com` no `SOC-SERVER`, `WIN-ENDPOINT`, `LINUX-ENDPOINT` e `KALI-ATTACKER`, com os endereços da rede `10.10.10.0/24`. O Dashboard em tema claro identifica a versão observada `v4.12.0` e três agentes.*

As imagens 06 e 07 são composições de evidências. Os resultados devem ser interpretados como registros das execuções mostradas, e não como prova de que a conectividade permanece disponível fora do horário da captura.

### 5.4 Portas de Comunicação

| Port | Protocol | Purpose |
|---|---|---|
| 1514 | TCP | Comunicação dos agentes Wazuh com o Manager |
| 1515 | TCP | Registro e autenticação de novos agentes |
| 443 | TCP | Acesso HTTPS ao Wazuh Dashboard, conforme configuração |
| 22 | TCP | Administração remota via SSH |
| 55000 | TCP | Wazuh Server API, quando habilitada e utilizada |

**Network Security Considerations:**
- A comunicação entre agentes e servidor SIEM é realizada por meio das portas necessárias ao funcionamento do Wazuh.
- O acesso administrativo é restrito aos sistemas autorizados.
- O dashboard não é exposto diretamente à internet.
- A comunicação entre as máquinas virtuais permanece dentro do escopo da rede do laboratório.

---

## 6. Fluxo de Dados

O fluxo de dados representa o caminho percorrido pelos eventos desde sua geração nos endpoints até sua disponibilização na plataforma de monitoramento.

### 6.1 Processo de Coleta de Eventos

1. **Event Generation:** Os sistemas Windows e Linux geram eventos durante suas atividades operacionais e de segurança.
2. **Log Collection:** Os agentes Wazuh instalados nos endpoints coletam os eventos configurados para monitoramento.
3. **Event Transmission:** Os eventos são encaminhados ao servidor central por meio da rede virtual, utilizando o canal de comunicação configurado.
4. **Event Analysis:** O Wazuh Manager recebe os eventos, realiza o processamento e aplica regras de análise e detecção.
5. **Alert Generation:** Eventos que correspondem às condições definidas pelas regras podem gerar alertas de segurança.
6. **Event Indexing:** Os alertas são encaminhados ao Wazuh Indexer para armazenamento e consulta.
7. **Security Monitoring:** Os dados ficam disponíveis no Wazuh Dashboard para visualização, análise e investigação.

### 6.2 Fluxo de Dados Simplificado

```text
+---------------------+
|  Windows Endpoint   |
|    WIN-ENDPOINT     |
+---------------------+
          |
          | Wazuh Agent
          |
          v
+---------------------+       +---------------------+
|  Ubuntu Endpoint    |       |    Kali Linux       |
|   LINUX-ENDPOINT    |       |   KALI-ATTACKER     |
+---------------------+       +---------------------+
          |                             |
          | Wazuh Agent                 | Controlled
          |                             | Security Tests
          v                             v
+---------------------------------------------------+
|                   SOC-LAB                         |
|                                                   |
|  +---------------------------------------------+  |
|  |               Wazuh Manager                 |  |
|  |       Event Collection & Analysis           |  |
|  +---------------------------------------------+  |
|                       |                           |
|                       v                           |
|  +---------------------------------------------+  |
|  |               Wazuh Indexer                 |  |
|  |          Alert Storage & Search             |  |
|  +---------------------------------------------+  |
|                       |                           |
|                       v                           |
|  +---------------------------------------------+  |
|  |              Wazuh Dashboard                |  |
|  |          Security Monitoring                |  |
|  +---------------------------------------------+  |
+---------------------------------------------------+

O Kali Linux é utilizado para gerar atividades controladas. Os eventos dessas atividades podem ser observados nos endpoints monitorados quando houver comunicação ou interação entre os sistemas e quando as fontes de eventos correspondentes estiverem configuradas.

```
---

## 7. Escopo de Segurança

O laboratório foi projetado para executar atividades de monitoramento e testes de segurança em um ambiente controlado.

### 7.1 Limites de Segurança
- Ambiente virtualizado e separado da infraestrutura de produção.
- Rede interna dedicada à comunicação entre as máquinas virtuais.
- Testes limitados aos sistemas pertencentes ao laboratório.
- Atividades de simulação executadas de maneira controlada.
- Acesso administrativo restrito aos responsáveis pelo ambiente.
- Dashboard sem exposição direta à internet.
- Comunicação externa utilizada somente quando necessária para atualizações e instalação de software.

### 7.2 Escopo das Atividades
As atividades contempladas incluem:
- Monitoramento de endpoints Windows e Linux.
- Análise de eventos de autenticação.
- Identificação de tentativas de acesso suspeitas.
- Monitoramento de atividades administrativas.
- Simulação de eventos relacionados à segurança.
- Análise de logs e alertas.
- Investigação de atividades suspeitas.
- Validação de regras de detecção.
- Monitoramento de vulnerabilidades, conforme os recursos habilitados.

### 7.3 Limitações
Este laboratório representa um ambiente controlado de aprendizagem e demonstração técnica. 

Sua infraestrutura não possui a escala, redundância, disponibilidade ou volume de eventos encontrados em um SOC corporativo de produção. 

O objetivo é reproduzir componentes, fluxos e procedimentos técnicos relevantes para operações de segurança, permitindo a execução de atividades práticas de monitoramento e investigação.

---

## 8. Validação

A validação da infraestrutura é realizada por meio de testes práticos destinados a comprovar a comunicação entre os componentes e o funcionamento da coleta centralizada de eventos.

### 8.1 Validação da Infraestrutura

| Validation | Expected Result |
|---|---|
| Wazuh Server installation | Serviços operacionais |
| Wazuh Dashboard access | Interface acessível |
| Wazuh Indexer status | Serviço operacional |
| Windows Agent integration | Agente conectado |
| Linux Agent integration | Agente conectado |
| Agent communication | Eventos recebidos pelo servidor |
| Event collection | Logs disponíveis para análise |
| Alert generation | Alertas gerados conforme regras aplicáveis |
| Network connectivity | Comunicação conforme topologia definida |
| Security testing | Atividades controladas observáveis nos endpoints |

### 8.2 Evidências de Validação
As evidências técnicas da arquitetura estão organizadas em `01-Architecture/imagens/`:

| Arquivo | Evidência documentada |
|---|---|
| `Imagem 01 — SOC Architecture.png` | Visão geral da arquitetura |
| `Imagem 02 — Network Topology.png` | Topologia lógica e fluxo de comunicação |
| `Imagem 03 — Wazuh Server.png` | Captura histórica do Dashboard; mostra a interface, mas não agentes ou eventos atuais |
| `Imagem 04 — Lab Environment.jpg` | Captura histórica das VMs; não é a evidência operacional atual |
| `Imagem 05 — interfaces configuradas no VirtualBox.jpg` | Adaptadores `SOC-LAB`, NAT e Host-Only |
| `Imagem 06 —Laboratório SOC com Quatro Máquinas Virtuais.png` | Quatro VMs ligadas, IPs, rotas, ping interno, TCP `1514` e agentes ativos |
| `Imagem 07 —Infográfico do Laboratório SOC-LAB Conectado.png` | Testes de NAT, DNS, versões observadas e Dashboard em tema claro |

As imagens 01 e 02 são diagramas de arquitetura. As imagens 03, 05, 06 e 07 são registros visuais de execuções ou estados observados, com as limitações descritas nas respectivas legendas.

### 8.3 Critérios de Sucesso
A infraestrutura é considerada operacional quando:
- O servidor Wazuh está funcional.
- O Wazuh Dashboard está acessível.
- Os agentes Windows e Linux estão conectados.
- Os eventos dos endpoints são recebidos pelo servidor.
- As regras de detecção processam os eventos correspondentes.
- Os alertas podem ser consultados e investigados no dashboard.
- As atividades de teste produzem evidências observáveis no ambiente.

---

## Estrutura do Projeto

```text
SOC-HOME-LAB/
│
├── 01-Architecture/
│   ├── README.md
│   └── imagens/
│       ├── Imagem 01 — SOC Architecture.png
│       ├── Imagem 02 — Network Topology.png
│       ├── Imagem 03 — Wazuh Server.png
│       ├── Imagem 04 — Lab Environment.jpg
│       ├── Imagem 05 — interfaces configuradas no VirtualBox.jpg
│       ├── Imagem 06 —Laboratório SOC com Quatro Máquinas Virtuais.png
│       └── Imagem 07 —Infográfico do Laboratório SOC-LAB Conectado.png
│
├── 02-Windows-Monitoring/
├── 03-Linux-Monitoring/
├── 04-Brute-Force-Detection/
├── 05-Privilege-Escalation/
├── 06-Suspicious-Processes/
├── 07-Incident-Investigation/
└── 08-Incident-Reports/
```
---

Project: SOC Home Lab — Security Monitoring & Threat Detection
Focus: Security Monitoring | SIEM | Log Analysis | Threat Detection | Incident Investigation | SOC Operations
