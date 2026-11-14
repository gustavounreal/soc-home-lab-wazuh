# 03 — Linux Monitoring

![Status](https://img.shields.io/badge/Status-Em_Desenvolvimento-yellow)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-blue)
![Agent](https://img.shields.io/badge/Agent-Wazuh_4.x-blue)
![OS](https://img.shields.io/badge/OS-Ubuntu_Server_22.04_LTS-E95420?logo=ubuntu)
![Hypervisor](https://img.shields.io/badge/Hypervisor-VirtualBox_7.x-orange)
![Network](https://img.shields.io/badge/Network-Host--Only-lightgrey)
![Monitoring](https://img.shields.io/badge/Monitoring-SSH%20%7C%20Sudo%20%7C%20FIM-green)
![Security](https://img.shields.io/badge/Security-Log_Analysis-red)

---

## 📋 Índice

- [1. Objetivos](#1-objetivos)
- [2. Ambiente](#2-ambiente)
- [3. Implantação do Wazuh Agent](#3-implantação-do-wazuh-agent)
- [4. Coleta de Logs](#4-coleta-de-logs)
- [5. Monitoramento de Autenticação SSH](#5-monitoramento-de-autenticação-ssh)
- [6. Gerenciamento de Usuários e Privilégios](#6-gerenciamento-de-usuários-e-privilégios)
- [7. Monitoramento de Processos e Serviços](#7-monitoramento-de-processos-e-serviços)
- [8. File Integrity Monitoring (FIM)](#8-file-integrity-monitoring-fim)
- [9. Eventos de Segurança Identificados](#9-eventos-de-segurança-identificados)
- [10. Matriz de Validação](#10-matriz-de-validação)
- [11. Troubleshooting](#11-troubleshooting)
- [12. Estrutura do Diretório](#12-estrutura-do-diretório)
- [13. Resultado do Laboratório](#13-resultado-do-laboratório)
- [14. Referências](#14-referências)

---

## 📖 Sobre o Projeto

Laboratório de monitoramento e auditoria de um servidor Linux com centralização de eventos no **Wazuh SIEM**.

O objetivo é demonstrar, na prática, a coleta e análise de eventos de segurança em um ambiente Linux, incluindo autenticações SSH, gerenciamento de usuários, atividades privilegiadas via `sudo`, monitoramento de processos e serviços e alterações na integridade de arquivos.

Este diretório faz parte de um laboratório integrado de monitoramento e segurança. O servidor Linux utiliza o mesmo Wazuh Manager configurado no cenário **Windows Monitoring**, permitindo centralizar eventos de diferentes sistemas operacionais em uma única plataforma.

---

## 1. Objetivos

Implementar e validar o monitoramento de um servidor Linux, demonstrando a capacidade de coletar, centralizar e analisar eventos relevantes para a segurança da informação.

Os principais objetivos são:

- ✅ Monitorar autenticações SSH bem-sucedidas e malsucedidas.
- ✅ Identificar atividades relacionadas à criação, alteração e remoção de usuários.
- ✅ Registrar a execução de comandos com privilégios administrativos via `sudo`.
- ✅ Acompanhar processos e serviços do sistema.
- ✅ Monitorar alterações em arquivos e diretórios por meio de File Integrity Monitoring (FIM).
- ✅ Centralizar os eventos no Wazuh Manager.
- ✅ Analisar os registros coletados e identificar atividades que merecem investigação.

O resultado esperado é a visualização dos eventos no Wazuh Dashboard, com possibilidade de relacioná-los aos registros originais do servidor Linux.

---

## 2. Ambiente

| Componente | Especificação |
|---|---|
| **Sistema monitorado** | Ubuntu Server 22.04 LTS |
| **Hostname** | `srv-linux01` |
| **Endereço IP** | `192.168.56.20` |
| **Wazuh Manager** | Ubuntu Server 22.04 LTS |
| **IP do Wazuh Manager** | `192.168.56.10` |
| **Wazuh** | Versão 4.x |
| **Wazuh Agent** | Versão 4.x |
| **Hipervisor** | VirtualBox 7.x |
| **Rede** | Host-only |
| **Usuário administrativo** | `labadmin` |

O ambiente utiliza uma rede isolada do tipo Host-only no VirtualBox. O Wazuh Manager foi instalado previamente no laboratório de Windows Monitoring e é reaproveitado neste cenário.

Essa arquitetura permite monitorar diferentes sistemas operacionais sem a necessidade de implantar um servidor SIEM independente para cada máquina.

### 🏗️ Arquitetura do Laboratório

```text
┌─────────────────────────────────────────────────────────────┐
│                    Rede Host-Only (192.168.56.0/24)         │
│                                                             │
│  ┌──────────────────┐         ┌──────────────────────────┐  │
│  │  Wazuh Manager   │◄────────│   srv-linux01 (Linux)    │  │
│  │  192.168.56.10   │  1514   │     192.168.56.20        │  │
│  │                  │         │   Wazuh Agent 4.x        │  │
│  │  Dashboard       │         │   Ubuntu Server 22.04    │  │
│  │  Indexer         │         └──────────────────────────┘  │
│  └──────────────────┘                                       │
│           ▲                                                 │
│           │                                                 │
│  ┌────────┴─────────┐                                       │
│  │  WKS-01 (Windows)│                                       │
│  │  Wazuh Agent     │                                       │
│  └──────────────────┘                                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Implantação do Wazuh Agent

O Wazuh Agent foi instalado no servidor `srv-linux01` utilizando o repositório oficial do projeto.

A instalação foi configurada para estabelecer comunicação com o Wazuh Manager no endereço `192.168.56.10`.

### 3.1. Configuração do Repositório

Comandos executados no servidor Linux:

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --no-default-keyring \
  --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import

sudo chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | \
  sudo tee /etc/apt/sources.list.d/wazuh.list

sudo apt update
```

### 3.2. Instalação do Agente

```bash
sudo WAZUH_MANAGER='192.168.56.10' WAZUH_AGENT_NAME='srv-linux01' \
  apt install -y wazuh-agent
```

Habilitação e inicialização do serviço:

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

### 3.3. Verificação

```bash
sudo systemctl status wazuh-agent
sudo /var/ossec/bin/wazuh-control status
```

A validação foi realizada em duas etapas:

1. Verificação local do serviço e dos processos do agente.
2. Verificação do estado de comunicação no Wazuh Dashboard.

No painel, o agente `srv-linux01` deve aparecer em **Agents Management** com o status `Active`.

**Evidências:**

- `images/01-linux-agent.png`
- `images/02-agent-dashboard.png`

---

## 4. Coleta de Logs

A coleta de eventos foi configurada no arquivo:

```bash
/var/ossec/etc/ossec.conf
```

Foram definidos arquivos de log relevantes para o monitoramento do sistema:

- `/var/log/auth.log`
- `/var/log/syslog`

Também foi configurada a coleta periódica da listagem de processos por meio do comando `ps aux`.

### 4.1. Configuração de Coleta

Blocos utilizados na configuração do agente:

```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>
</localfile>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/syslog</location>
</localfile>

<localfile>
  <log_format>full_command</log_format>
  <command>ps aux</command>
  <frequency>300</frequency>
</localfile>
```

O arquivo `auth.log` é utilizado como fonte principal para eventos de autenticação, atividades relacionadas ao `sudo` e determinados registros de gerenciamento de usuários.

O arquivo `syslog` complementa a coleta com registros de serviços e atividades do sistema.

A coleta de `ps aux` ocorre a cada 300 segundos, permitindo acompanhar periodicamente os processos em execução.

> ⚠️ **Nota:** Os blocos devem ser incorporados à configuração existente do agente, preservando as demais configurações necessárias.

### 4.2. Reinicialização do Agente

Após a alteração:

```bash
sudo systemctl restart wazuh-agent
```

A validação foi realizada no Wazuh Dashboard, filtrando os eventos pelo agente:

```text
agent.name: srv-linux01
```

---

## 5. Monitoramento de Autenticação SSH

O monitoramento de autenticação SSH permite identificar tentativas de acesso ao servidor, tanto bem-sucedidas quanto malsucedidas.

O serviço SSH estava ativo no servidor `srv-linux01`.

Os testes foram realizados a partir de máquinas pertencentes ao ambiente de laboratório, incluindo a estação `WKS-01` (Windows 11) e o próprio Wazuh Manager.

### 5.1. Autenticação Bem-Sucedida

Comando executado:

```bash
ssh labadmin@192.168.56.20
```

Registro correspondente observado em `/var/log/auth.log`:

```text
sshd[1234]: Accepted password for labadmin from 192.168.56.10 port 51422 ssh2
```

O evento representa uma autenticação SSH realizada com sucesso.

### 5.2. Autenticação Malsucedida

Foi realizada uma tentativa controlada de acesso utilizando uma senha incorreta, repetida três vezes.

Comando:

```bash
ssh labadmin@192.168.56.20
```

Registros observados:

```text
sshd[1240]: Failed password for labadmin from 192.168.56.10 port 51430 ssh2
sshd[1241]: Failed password for labadmin from 192.168.56.10 port 51432 ssh2
sshd[1242]: Failed password for labadmin from 192.168.56.10 port 51434 ssh2
```

Os registros permitem identificar:

- Usuário utilizado na tentativa.
- Endereço IP de origem.
- Horário da ocorrência.
- Resultado da autenticação.
- Serviço responsável pelo registro.

### 5.3. Identificação no Wazuh

Os eventos são analisados pelo mecanismo de regras do Wazuh.

Entre as regras relacionadas ao cenário estão:

| Regra | Descrição |
|---|---|
| `5710` | Falha de autenticação SSH |
| `5715` | Autenticação SSH bem-sucedida |
| `5720` | Detecção relacionada a múltiplas falhas de autenticação SSH |

A regra de detecção de força bruta depende dos critérios de correlação configurados, como quantidade de ocorrências e intervalo de tempo.

Portanto, três falhas isoladas não devem ser consideradas, por si só, garantia de disparo da regra `5720`. A validação deve considerar a configuração efetiva da regra e os eventos apresentados no dashboard.

**Evidências:**

- `images/03-authentication-logs.png`
- `images/04-failed-login.png`

---

## 6. Gerenciamento de Usuários e Privilégios

O monitoramento de contas permite identificar alterações que podem representar atividades administrativas legítimas ou comportamentos suspeitos.

Foram realizadas operações controladas de criação, alteração e remoção de usuários, além da execução de comandos com privilégios administrativos.

### 6.1. Criação de Usuário

Comandos:

```bash
sudo useradd -m -s /bin/bash labuser01
sudo passwd labuser01
```

Registros observados:

```text
useradd[1301]: new user: name=labuser01, UID=1001, GID=1001, home=/home/labuser01, shell=/bin/bash
passwd[1305]: pam_unix(passwd:chauthtok): password changed for labuser01
```

### 6.2. Alteração de Grupo

Foi realizada a inclusão do usuário de teste no grupo administrativo `sudo`.

```bash
sudo usermod -aG sudo labuser01
```

Registro observado:

```text
usermod[1310]: add 'labuser01' to group 'sudo'
```

Essa alteração merece atenção durante uma investigação, pois a inclusão de uma conta em um grupo privilegiado pode ampliar seus níveis de acesso.

### 6.3. Remoção de Usuário

Comando:

```bash
sudo userdel -r labuser01
```

Registro observado:

```text
userdel[1315]: delete user 'labuser01'
```

### 6.4. Monitoramento de Atividades Sudo

Foram executados comandos administrativos utilizando o usuário `labadmin`.

```bash
sudo cat /etc/shadow
sudo systemctl restart ssh
```

Registros observados:

```text
sudo[1320]: labadmin : TTY=pts/0 ; PWD=/home/labadmin ; USER=root ; COMMAND=/usr/bin/cat /etc/shadow

sudo[1322]: labadmin : TTY=pts/0 ; PWD=/home/labadmin ; USER=root ; COMMAND=/usr/bin/systemctl restart ssh
```

Esses registros permitem identificar:

- Usuário que iniciou a operação.
- Usuário efetivo utilizado na execução.
- Terminal de origem.
- Diretório de trabalho.
- Comando executado.

### 6.5. Identificação no Wazuh

Entre as regras relacionadas às atividades de gerenciamento de usuários e privilégios estão:

| Regra | Descrição |
|---|---|
| `5401` | Tentativa de execução de sudo sem sucesso, quando aplicável |
| `5402` | Execução de comando sudo com sucesso |
| `5902` | Evento relacionado à criação de usuário |

As regras efetivamente acionadas dependem da versão do ruleset, dos registros recebidos e da configuração do ambiente.

**Evidências:**

- `images/05-user-management.png`
- `images/06-sudo-activity.png`

---

## 7. Monitoramento de Processos e Serviços

O monitoramento de processos e serviços permite acompanhar atividades executadas no sistema e identificar mudanças relevantes em seu funcionamento.

Além da coleta periódica de `ps aux`, foram realizadas verificações do estado dos serviços SSH e Wazuh Agent.

### 7.1. Verificação de Serviços

Comandos utilizados:

```bash
systemctl status ssh
systemctl status wazuh-agent
```

Saída resumida observada:

```text
ssh.service - OpenBSD Secure Shell server
     Active: active (running)

wazuh-agent.service - Wazuh agent
     Active: active (running)
```

### 7.2. Simulação de Alteração de Estado

Foi realizada uma parada controlada e posterior inicialização do serviço SSH.

```bash
sudo systemctl stop ssh
sudo systemctl start ssh
```

A atividade permite observar registros relacionados à alteração de estado do serviço.

Os eventos podem ser identificados nos registros do sistema e encaminhados ao Wazuh, conforme a configuração de coleta.

A análise deve considerar o horário da ocorrência, o serviço afetado e o contexto da operação.

**Evidência:**

- Parte de `images/07-system-monitoring.png`

---

## 8. File Integrity Monitoring (FIM)

O File Integrity Monitoring permite identificar alterações em arquivos e diretórios monitorados.

Neste laboratório, o FIM foi configurado para acompanhar:

- `/etc`
- `/opt/lab-fim`

O diretório `/etc` contém arquivos importantes para a configuração do sistema operacional e de seus serviços.

O diretório `/opt/lab-fim` foi utilizado como área controlada para os testes de integridade.

### 8.1. Configuração

Bloco utilizado no arquivo `ossec.conf`:

```xml
<syscheck>
  <frequency>300</frequency>
  <directories check_all="yes">/etc</directories>
  <directories check_all="yes">/opt/lab-fim</directories>
</syscheck>
```

A frequência de verificação foi definida em 300 segundos.

### 8.2. Simulação de Alteração

Foi criado um diretório de teste e realizada a criação e posterior modificação de um arquivo.

```bash
sudo mkdir -p /opt/lab-fim

echo "conteudo inicial" | sudo tee /opt/lab-fim/teste.txt

echo "conteudo alterado" | sudo tee /opt/lab-fim/teste.txt
```

### 8.3. Análise dos Eventos

No Wazuh Dashboard, os eventos podem ser consultados na área de **File Integrity Monitoring**.

Entre as regras relacionadas estão:

| Regra | Descrição |
|---|---|
| `550` | Arquivo modificado |
| `554` | Arquivo adicionado |

Os eventos de FIM podem apresentar informações como:

- Caminho do arquivo.
- Tipo de alteração.
- Data e horário.
- Hash do arquivo.
- Informações anteriores e posteriores à alteração, conforme os dados disponíveis.

A análise permite identificar modificações em arquivos monitorados e avaliar se a atividade corresponde a uma alteração administrativa esperada ou a um comportamento que exige investigação.

**Evidência:**

- Parte de `images/07-system-monitoring.png`

---

## 9. Eventos de Segurança Identificados

A tabela apresenta os principais registros utilizados durante o laboratório.

| Evento / Registro | Finalidade | Origem |
|---|---|---|
| `/var/log/auth.log` | Autenticação e atividades administrativas | `srv-linux01` |
| `Failed password` | Identificação de falhas de autenticação SSH | `auth.log` |
| `Accepted password` | Identificação de autenticações SSH bem-sucedidas | `auth.log` |
| `sudo` | Registro de comandos privilegiados | `auth.log` |
| `useradd` | Criação de usuários | `auth.log` |
| `usermod` | Alteração de contas e grupos | `auth.log` |
| `userdel` | Remoção de usuários | `auth.log` |
| FIM | Identificação de alterações em arquivos monitorados | Wazuh |
| Mudança de estado de serviço | Monitoramento de serviços do sistema | `syslog` e Wazuh |
| `ps aux` | Coleta periódica de processos em execução | Wazuh Agent |

---

## 10. Matriz de Validação

Os testes abaixo foram utilizados para validar a coleta e a visualização dos eventos no ambiente.

| Teste | Comando ou Ação | Resultado Esperado | Resultado Observado |
|---|---|---|---|
| Agente conectado | `systemctl status wazuh-agent` | Serviço ativo e agente conectado ao Manager | Serviço ativo; agente `srv-linux01` com status `Active` no dashboard |
| Login SSH bem-sucedido | `ssh labadmin@192.168.56.20` | Registro `Accepted password` | Evento presente em `auth.log` e no Wazuh, regra `5715` |
| Login SSH malsucedido | Senha incorreta repetida três vezes | Registros `Failed password` | Três eventos registrados em `auth.log` e encaminhados ao Wazuh |
| Criação de usuário | `useradd` e `passwd` | Registro de criação e alteração de senha | Eventos presentes em `auth.log` e no Wazuh |
| Alteração de grupo | `usermod -aG sudo` | Registro de inclusão no grupo administrativo | Evento presente em `auth.log` e no Wazuh |
| Remoção de usuário | `userdel -r` | Registro de remoção | Evento presente em `auth.log` e no Wazuh |
| Atividade sudo | `sudo cat /etc/shadow` | Registro de execução privilegiada | Evento presente em `auth.log` e no Wazuh |
| Alteração de serviço | `systemctl stop/start ssh` | Registros de alteração do serviço | Eventos presentes nos registros do sistema e encaminhados ao Wazuh |
| FIM | Criação e alteração de arquivo em `/opt/lab-fim` | Identificação de inclusão e modificação | Eventos apresentados no Wazuh, regras `550` e `554`, conforme a operação |

A matriz registra os resultados observados durante a execução dos testes. O acionamento de regras específicas deve ser interpretado conforme a versão do Wazuh e as configurações efetivamente utilizadas.

---

## 11. Troubleshooting

### 11.1. Agente Apresentado como Disconnected

**Sintoma:**

Após a instalação, o agente não aparecia como `Active` no dashboard do Wazuh.

**Verificações realizadas:**

- Estado do serviço `wazuh-agent`.
- Configuração do endereço do Wazuh Manager.
- Conectividade entre o agente e o Manager.
- Regras de firewall.
- Disponibilidade da porta de comunicação.

**Ponto importante sobre a comunicação:**

Por padrão, o agente inicia a comunicação com o Manager utilizando TCP na porta `1514`.

Portanto, a liberação deve considerar o sentido efetivo da conexão:

- No firewall do Manager, permitir a entrada da comunicação originada pelo agente na porta `1514/TCP`.
- Se o firewall do agente restringir conexões de saída, permitir a comunicação de saída para `192.168.56.10:1514/TCP`.

A porta `1515/TCP` é normalmente utilizada para o serviço de registro de agentes, quando esse mecanismo está habilitado.

**Exemplo de liberação no firewall do Manager, caso o UFW esteja ativo:**

```bash
sudo ufw allow from 192.168.56.20 to any port 1514 proto tcp
```

Após os ajustes, verificar novamente o serviço e o estado do agente no dashboard.

> ⚠️ **Importante:** A regra de firewall deve ser aplicada na máquina e no sentido correspondentes à configuração real da rede. Não se deve liberar indiscriminadamente portas de entrada no servidor monitorado.

### 11.2. Eventos Não Aparecem no Wazuh

Verificações recomendadas:

```bash
sudo systemctl status wazuh-agent

sudo tail -f /var/ossec/logs/ossec.log

sudo tail -f /var/log/auth.log
```

Também é necessário verificar:

- Se o caminho do arquivo de log está correto.
- Se o bloco `<localfile>` está configurado adequadamente.
- Se o XML do `ossec.conf` está válido.
- Se o agente foi reiniciado após a alteração.
- Se o Manager está recebendo os eventos.
- Se o filtro utilizado no dashboard corresponde ao agente correto.

---

## 12. Estrutura do Diretório

```text
03-Linux-Monitoring/
│
├── README.md
│
├── images/
│   ├── 01-linux-agent.png
│   ├── 02-agent-dashboard.png
│   ├── 03-authentication-logs.png
│   ├── 04-failed-login.png
│   ├── 05-user-management.png
│   ├── 06-sudo-activity.png
│   └── 07-system-monitoring.png
│
└── scripts/
    ├── install-wazuh-agent.sh
    ├── ossec-localfile.conf
    └── ossec-syscheck.conf
```

As imagens devem representar capturas reais do ambiente, enquanto os arquivos da pasta `scripts/` devem conter os comandos e blocos de configuração efetivamente utilizados.

---

## 13. Resultado do Laboratório

O cenário demonstrou a implantação de um agente Wazuh em um servidor Ubuntu Server e a centralização de eventos Linux em uma plataforma SIEM compartilhada com o ambiente Windows.

Foram trabalhadas atividades relacionadas a:

- ✅ Monitoramento de autenticação SSH.
- ✅ Identificação de tentativas de acesso malsucedidas.
- ✅ Rastreamento de alterações em contas de usuário.
- ✅ Monitoramento de comandos privilegiados.
- ✅ Acompanhamento de processos e serviços.
- ✅ Monitoramento de integridade de arquivos.
- ✅ Consulta e análise de eventos centralizados no Wazuh.
- ✅ Verificação de conectividade e resolução de problemas do agente.

O laboratório fornece uma base prática para atividades de monitoramento e análise de eventos em sistemas Linux, competências relevantes para rotinas de infraestrutura, segurança da informação e operações SOC.

As atividades foram executadas em ambiente controlado, com usuários e operações de teste.

---

## 14. Referências

- [Wazuh — Documentação oficial](https://documentation.wazuh.com/)
- [Wazuh — Linux Endpoint Security](https://documentation.wazuh.com/current/user-manual/capabilities/log-data-collection/index.html)
- [Wazuh — File Integrity Monitoring](https://documentation.wazuh.com/current/user-manual/capabilities/file-integrity/index.html)
- [Wazuh — Ruleset](https://documentation.wazuh.com/current/user-manual/ruleset/index.html)
- [Ubuntu Server — Documentação oficial](https://documentation.ubuntu.com/server/)
- [OpenSSH — Manual oficial](https://www.openssh.com/manual.html)

---

<div align="center">

**Projeto:** SOC Laboratory  
**Módulo:** 03 — Linux Monitoring  
**SIEM:** Wazuh  
**Sistema Monitorado:** Ubuntu Server 22.04 LTS

---

⌨️ Desenvolvido como parte do laboratório SOC

</div>