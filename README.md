# SOC Home Lab | Blue Team & Incident Detection

![Status](https://img.shields.io/badge/Status-Em_Desenvolvimento-yellow)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-blue)
![VirtualBox](https://img.shields.io/badge/Hypervisor-VirtualBox_7.x-orange)
![Windows](https://img.shields.io/badge/OS-Windows_11_Pro-0078D6?logo=windows)
![Linux](https://img.shields.io/badge/OS-Ubuntu_24.04_LTS-E95420?logo=ubuntu)
![Kali](https://img.shields.io/badge/OS-Kali_Linux-557C94?logo=kali-linux)

## Sobre o Projeto

Este repositório contém a documentação e as implementações práticas de um laboratório integrado de SOC (Security Operations Center) N1. O objetivo principal não é construir uma infraestrutura complexa, mas sim demonstrar, na prática, que possuo as habilidades necessárias para executar o fluxo de trabalho real de um analista de segurança: gerar um evento, coletar os registros, detectar uma atividade suspeita, investigar e documentar a ocorrência.

O laboratório foi dividido em **cenários de investigação independentes**, todos rodando dentro do mesmo ambiente controlado.

## Arquitetura do Ambiente

A estrutura foi desenhada para ser enxuta e funcional, focada na detecção e resposta:

- **Wazuh Server:** Plataforma SIEM para centralização dos eventos, correlação e análise.
- **Windows 10/11:** Máquina monitorada (Endpoint).
- **Ubuntu Linux:** Máquina monitorada (Endpoint).
- **Kali Linux:** Máquina utilizada para simulação controlada de ataques (Threat Emulation).
- **Wazuh Dashboard:** Interface para visualização, triagem e investigação dos alertas.

*Para visualizar o diagrama completo da arquitetura, topologia de rede e detalhes de infraestrutura, acesse a pasta [`01-Architecture`](./01-Architecture).*

## Divisão dos Cenários (Módulos)

Cada diretório neste repositório representa um módulo específico de atuação do SOC, focado em uma competência técnica.

| Diretório | Cenário Investigado | Competência Demonstrada |
|---|---|---|
| [`01-Architecture`](./01-Architecture) | Infraestrutura e Topologia | Planejamento e Virtualização |
| [`02-Windows-Monitoring`](./02-Windows-Monitoring) | Coleta de Logs Windows | Integração de Agentes e Logs |
| [`03-Linux-Monitoring`](./03-Linux-Monitoring) | Coleta de Logs Linux | Auditoria de Sistema e Autenticação |
| [`04-Brute-Force-Detection`](./04-Brute-Force-Detection) | Falhas repetidas de autenticação | Detecção de Brute Force |
| [`05-Privilege-Escalation`](./05-Privilege-Escalation) | Alteração de privilégios | Detecção de Elevação de Privilégios |
| [`06-Suspicious-Processes`](./06-Suspicious-Processes) | Execução de processos suspeitos | Análise de Comportamento Anômalo |
| [`07-Incident-Investigation`](./07-Incident-Investigation) | Investigação de alertas | Triagem, Linha do Tempo e Classificação |
| [`08-Incident-Reports`](./08-Incident-Reports) | Resposta a incidentes | Documentação, Relatórios e Escalonamento |

## Metodologia de Trabalho

O foco deste portfólio é a qualidade da investigação e não a quantidade de simulações superficiais. Para cada cenário documentado neste repositório, a seguinte estrutura de análise é aplicada:

1. **Simulação:** Qual atividade controlada foi executada (ex: via Kali Linux).
2. **Geração de Evento:** Qual log ou evento de segurança foi gerado no endpoint.
3. **Detecção (SIEM):** Como o Wazuh identificou o evento e qual regra foi acionada.
4. **Evidência:** Quais foram as evidências técnicas coletadas (screenshots, logs, IOCs).
5. **Classificação:** Como o alerta foi classificado (Falso Positivo, Suspeita ou Incidente).
6. **Ação de Resposta:** Qual seria a ação recomendada ou o escalonamento adequado.

## Estratégia de Desenvolvimento

O laboratório é construído de forma progressiva. A prioridade atual está em executar três cenários com excelência antes de expandir a infraestrutura:
- *Brute force* com detecção e investigação completa.
- Criação de usuário e alteração de privilégios.
- Execução de processo suspeito e análise profunda do alerta.

---
**Foco:** Security Monitoring | SIEM (Wazuh) | Log Analysis | Threat Detection | Incident Investigation | SOC Operations
