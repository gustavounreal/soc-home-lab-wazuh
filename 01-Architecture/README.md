# SOC Home Lab — Architecture & Infrastructure

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

## 1. Overview

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

### Architecture Overview

<img width="1672" height="941" alt="Imagem 01 — SOC Architecture" src="https://github.com/user-attachments/assets/ef720929-7cd2-4e15-bb75-8c3282562ada" />

*Figura 01 — Visão geral da arquitetura do laboratório SOC.*

---

## 2. Architecture

A arquitetura foi organizada em três camadas principais, cada uma com responsabilidades específicas dentro do ambiente.

### 2.1 SIEM & Monitoring Layer
Camada responsável pela centralização, processamento, correlação e visualização dos eventos de segurança.

**Componente principal:** Wazuh Server.

**Responsabilidades:**
- Recebimento de eventos dos endpoints.
- Gerenciamento dos agentes Wazuh.
- Análise e classificação de eventos.
- Aplicação de regras de detecção.
- Geração e armazenamento de alertas.
- Disponibilização de dashboards de monitoramento.

### 2.2 Endpoint Monitoring Layer
Camada composta pelos sistemas operacionais monitorados, responsáveis pela geração e envio dos eventos de segurança.

**Componentes:** Windows Endpoint e Ubuntu Linux Endpoint.

**Responsabilidades:**
- Geração de eventos operacionais e de segurança.
- Registro de atividades de autenticação.
- Monitoramento de alterações no sistema.
- Coleta de logs.
- Envio de eventos ao servidor central.

### 2.3 Security Testing Layer
Camada destinada à execução controlada de atividades de reconhecimento, testes de segurança e geração de eventos para validação dos mecanismos de monitoramento.

**Componente:** Kali Linux.

**Responsabilidades:**
- Reconhecimento controlado do ambiente.
- Geração de atividades para testes.
- Simulação de tentativas de autenticação.
- Execução de testes de segurança autorizados.
- Validação da capacidade de observação e detecção do SIEM.

### Logical Architecture

<img width="1672" height="941" alt="Imagem 02 — Network Topology" src="https://github.com/user-attachments/assets/e50670e4-7a0e-4b11-9649-9c77db19aeb9" />

*Figura 02 — Topologia lógica e comunicação entre os componentes.*

---

## 3. Components

A infraestrutura é composta por quatro máquinas virtuais, distribuídas em duas redes distintas para separar o tráfego de monitoramento da interface de gerenciamento:

| Component | Operating System | IP (Internal Network) | Role |
|---|---|---|---|
| **SOC-SERVER** | Ubuntu Server 24.04 LTS | `10.10.10.10` | SIEM, gerenciamento centralizado e Dashboard |
| **WIN-ENDPOINT** | Windows 11 Pro | `10.10.10.20` | Endpoint monitorado (Windows) |
| **LINUX-ENDPOINT** | Ubuntu Server 24.04 LTS | `10.10.10.30` | Endpoint monitorado (Linux) |
| **KALI-ATTACKER** | Kali Linux | `10.10.10.40` | Estação de testes controlados de segurança |

### 3.1 Wazuh Server
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

<img width="1672" height="941" alt="Imagem 03 — Wazuh Server" src="https://github.com/user-attachments/assets/3d42d967-4b6e-45da-94f1-84e6aaff537b" />

*Figura 03 — Servidor Wazuh instalado e operacional.*

### 3.2 Windows Endpoint
Sistema Windows integrado ao ambiente de monitoramento por meio do Wazuh Agent.

**Fontes de eventos monitoradas:**
- Windows Event Logs.
- Eventos de autenticação.
- Tentativas de acesso.
- Alterações de usuários e grupos.
- Atividades relacionadas à segurança do sistema.
- Eventos de serviços e processos.

### 3.3 Ubuntu Linux Endpoint
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

*O Kali Linux atua como estação de testes e não depende de um agente Wazuh para desempenhar essa função.*

---

## 4. Infrastructure

O laboratório utiliza virtualização para executar os diferentes sistemas operacionais em um ambiente controlado, permitindo a integração dos componentes e a simulação de cenários de monitoramento de segurança.

### 4.1 Virtualization Environment

| Resource | Configuration |
|---|---|
| Hypervisor | Type 2 Hypervisor |
| Host Operating System | Windows 11 Pro 64-bit |
| Virtualization Platform | Oracle VM VirtualBox 7.x |
| Network Mode | Internal Network + NAT + Host-Only |
| Host RAM | 32 GB DDR4/DDR5 |
| Host Storage | 512 GB SSD |
| Virtual Machines | 4 |

### 4.2 Virtual Machine Resources

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

### 4.3 Lab Environment

<img width="1672" height="941" alt="Imagem 04 — Lab Environment" src="https://github.com/user-attachments/assets/5170a92c-57e4-4122-8fad-d5caf9335988" />

*Figura 04 — Visão geral das máquinas virtuais do laboratório.*

---

## 5. Network Topology

A comunicação entre os sistemas ocorre por meio de uma infraestrutura de rede virtual configurada no Oracle VM VirtualBox (`VirtualBox`). O ambiente utiliza uma rede interna dedicada para a comunicação entre os componentes e uma interface Host-Only para acesso administrativo seguro.

### 5.1 Network Configuration

- **Network Name:** SOC-LAB
- **Network Address:** 10.10.10.0/24
- **Subnet Mask:** 255.255.255.0

| Component | Hostname | IP Address (Internal) | IP Address (Host-Only) |
|---|---|---|---|
| Wazuh Server | SOC-SERVER | `10.10.10.10` | `192.168.56.10` |
| Windows Endpoint | WIN-ENDPOINT | `10.10.10.20` | — |
| Ubuntu Endpoint | LINUX-ENDPOINT | `10.10.10.30` | — |
| Kali Linux | KALI-ATTACKER | `10.10.10.40` | — |

### 5.2 Network Interfaces

| Interface | Configuration | Purpose |
|---|---|---|
| **Internal Network** | `SOC-LAB` (`10.10.10.0/24`) | Comunicação isolada e tráfego de monitoramento entre as VMs |
| **NAT** | VirtualBox NAT | Acesso externo para atualizações e instalação de pacotes nos nós |
| **Host-Only** | VirtualBox Host-Only (`192.168.56.0/24`) | Acesso seguro do host físico (Windows) ao Wazuh Dashboard (`192.168.56.10`) |

- O **SOC-SERVER** utiliza a interface Host-Only (`192.168.56.10`) para permitir o acesso ao Wazuh Dashboard diretamente pelo navegador do computador hospedeiro com total segurança.
- A interface `Internal Network` (`10.10.10.x`) gerencia o tráfego de agentes e testes entre o SOC, os endpoints e o Kali Linux.
- A interface NAT permite acesso externo quando necessário para atualização dos sistemas e instalação de dependências.

### 5.3 Communication Ports

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

## 6. Data Flow

O fluxo de dados representa o caminho percorrido pelos eventos desde sua geração nos endpoints até sua disponibilização na plataforma de monitoramento.

### 6.1 Event Collection Process

1. **Event Generation:** Os sistemas Windows e Linux geram eventos durante suas atividades operacionais e de segurança.
2. **Log Collection:** Os agentes Wazuh instalados nos endpoints coletam os eventos configurados para monitoramento.
3. **Event Transmission:** Os eventos são encaminhados ao servidor central por meio da rede virtual, utilizando o canal de comunicação configurado.
4. **Event Analysis:** O Wazuh Manager recebe os eventos, realiza o processamento e aplica regras de análise e detecção.
5. **Alert Generation:** Eventos que correspondem às condições definidas pelas regras podem gerar alertas de segurança.
6. **Event Indexing:** Os alertas são encaminhados ao Wazuh Indexer para armazenamento e consulta.
7. **Security Monitoring:** Os dados ficam disponíveis no Wazuh Dashboard para visualização, análise e investigação.

### 6.2 Simplified Data Flow

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

## 7. Security Scope

O laboratório foi projetado para executar atividades de monitoramento e testes de segurança em um ambiente controlado.

### 7.1 Security Boundaries
- Ambiente virtualizado e separado da infraestrutura de produção.
- Rede interna dedicada à comunicação entre as máquinas virtuais.
- Testes limitados aos sistemas pertencentes ao laboratório.
- Atividades de simulação executadas de maneira controlada.
- Acesso administrativo restrito aos responsáveis pelo ambiente.
- Dashboard sem exposição direta à internet.
- Comunicação externa utilizada somente quando necessária para atualizações e instalação de software.

### 7.2 Scope of Activities
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

### 7.3 Limitations
Este laboratório representa um ambiente controlado de aprendizagem e demonstração técnica. 

Sua infraestrutura não possui a escala, redundância, disponibilidade ou volume de eventos encontrados em um SOC corporativo de produção. 

O objetivo é reproduzir componentes, fluxos e procedimentos técnicos relevantes para operações de segurança, permitindo a execução de atividades práticas de monitoramento e investigação.

---

## 8. Validation

A validação da infraestrutura é realizada por meio de testes práticos destinados a comprovar a comunicação entre os componentes e o funcionamento da coleta centralizada de eventos.

### 8.1 Infrastructure Validation

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

### 8.2 Validation Evidence
As evidências técnicas são organizadas ao longo da documentação do projeto, incluindo:
- Capturas da infraestrutura virtualizada.
- Estado dos agentes conectados.
- Registros de eventos recebidos.
- Alertas gerados pelo Wazuh.
- Resultados dos testes de comunicação.
- Evidências de atividades de segurança simuladas.
- Registros de análise e investigação.

### 8.3 Success Criteria
A infraestrutura é considerada operacional quando:
- O servidor Wazuh está funcional.
- O Wazuh Dashboard está acessível.
- Os agentes Windows e Linux estão conectados.
- Os eventos dos endpoints são recebidos pelo servidor.
- As regras de detecção processam os eventos correspondentes.
- Os alertas podem ser consultados e investigados no dashboard.
- As atividades de teste produzem evidências observáveis no ambiente.

---

## Project Structure

```text
SOC-HOME-LAB/
│
├── 01-Architecture/
│   ├── README.md
│   └── images/
│       ├── 01-soc-architecture.png
│       ├── 02-network-topology.png
│       ├── 03-wazuh-server.png
│       └── 04-lab-environment.png
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
