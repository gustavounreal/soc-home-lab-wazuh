# SOC Home Lab — Suspicious Processes

![Status](https://img.shields.io/badge/Status-Em_Desenvolvimento-yellow)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-blue)
![Scope](https://img.shields.io/badge/Scope-SOC_N1-orange)

## Sobre o cenário

Este módulo documenta a análise de processos potencialmente suspeitos nos
endpoints do SOC Home Lab. O objetivo é demonstrar como um analista N1 coleta
contexto, valida a origem de um processo e evita classificar uma atividade
benigna como incidente sem evidência suficiente.

O cenário utiliza a arquitetura existente:

- `WIN-ENDPOINT` (`10.10.10.20`): Windows 11 Pro;
- `LINUX-ENDPOINT` (`10.10.10.30`): Ubuntu Server 24.04 LTS;
- `SOC-SERVER` (`10.10.10.10`): Wazuh Manager, Indexer e Dashboard;
- rede interna `SOC-LAB` (`10.10.10.0/24`);
- acesso ao Dashboard em `https://192.168.56.10`.

Não são adicionadas máquinas, EDRs, Sysmon, auditd ou ferramentas de análise de
malware.

## O que o módulo demonstra

- geração controlada de eventos de criação de processos no Windows;
- leitura de nome, caminho, usuário e linha de comando quando disponíveis;
- validação do processo pai e do contexto operacional;
- consulta de processos ativos no Linux;
- distinção entre processo de teste, atividade administrativa e comportamento
  realmente suspeito;
- triagem, investigação e recomendação de resposta.

## Limitações de coleta

No Windows, o script de auditoria atual habilita a criação de processos e gera
o evento `4688`. A disponibilidade de caminho, processo pai e linha de
comando depende da política de auditoria e dos campos coletados pelo agente.

No Linux, o agente coleta `ps aux` a cada 300 segundos. Essa é uma fotografia
periódica de processos, não um mecanismo de captura contínua de toda criação de
processo. Um processo curto pode terminar antes da próxima coleta.

## Fluxo técnico

```text
WIN-ENDPOINT (10.10.10.20)  --+
                              |
LINUX-ENDPOINT (10.10.10.30) -+--> SOC-SERVER (10.10.10.10)
                                    Wazuh Manager / Indexer / Dashboard

Host físico --> Host-Only (192.168.56.10) --> Dashboard
```

Os agentes enviam eventos pela rede interna `SOC-LAB` usando o canal
operacional do Wazuh em TCP `1514`. A rede Host-Only é utilizada somente pelo
host físico para acessar o Dashboard.

## Cenário Windows

O script
[`02-windows-command-audit.ps1`](../02-Windows-Monitoring/scripts/02-windows-command-audit.ps1)
habilita a auditoria de criação de processos e executa comandos de validação
controlados:

- `notepad.exe`;
- `cmd.exe /c echo SOC Home Lab Test`.

Esses comandos são benignos e servem para confirmar a coleta do Event ID
`4688`. A existência de um evento `4688` não significa, isoladamente, que o
processo seja malicioso.

Antes do teste:

1. confirme que o script será executado como Administrador no
   `WIN-ENDPOINT`;
2. confirme que o serviço `WazuhSvc` está ativo;
3. confirme que o agente está registrado como `WIN-ENDPOINT`;
4. anote o horário de início do teste.

Depois, localize no Windows Event Viewer e no Wazuh:

| Campo | Valor a confirmar |
|---|---|
| Event ID | `4688` |
| Host | `WIN-ENDPOINT` |
| IP do endpoint | `10.10.10.20` |
| Processo | nome real exibido no evento |
| Usuário | conta que iniciou o processo |
| Processo pai | somente se o campo estiver disponível |
| Linha de comando | somente se a auditoria estiver configurada para coletá-la |

Não preencha campos ausentes com valores presumidos.

## Cenário Linux

No `LINUX-ENDPOINT`, a coleta configurada é:

```xml
<localfile>
  <log_format>full_command</log_format>
  <command>ps aux</command>
  <frequency>300</frequency>
</localfile>
```

Para uma inspeção local no momento da análise:

```bash
ps -eo pid,ppid,user,%cpu,%mem,etime,cmd --sort=-%cpu | head -n 15
pstree -p
systemctl --type=service --state=running
```

O script
[`03-linux-agent-commands.sh`](../02-Windows-Monitoring/scripts/03-linux-agent-commands.sh)
também pode ser utilizado para validar o serviço do agente e observar
processos relacionados ao Wazuh e ao SSH. Ele é uma ferramenta de inspeção,
não um gerador de alerta de processo suspeito.

## Triagem N1

Para cada processo sinalizado, registre:

1. Qual endpoint executou o processo?
2. Qual foi o PID e, quando disponível, o PPID?
3. Qual usuário iniciou a execução?
4. Qual caminho completo e linha de comando foram observados?
5. O processo pai é esperado para aquele executável?
6. O horário coincide com uma tarefa administrativa ou teste?
7. O binário está em um caminho esperado e possui assinatura ou pacote
   conhecido?
8. Existem conexões de rede ou alterações de arquivo relacionadas?
9. Há repetição, persistência ou tentativa de evasão?

Classifique o resultado como:

- **Atividade autorizada:** processo esperado ou criado intencionalmente para
  validar a coleta;
- **Suspeita:** contexto incompleto ou combinação anômala que exige investigação;
- **Incidente:** evidência suficiente de execução não autorizada ou impacto.

Um `cmd.exe`, `powershell.exe`, `bash` ou `ssh` não é malicioso por si só. O
contexto, a linha de comando, o usuário, o processo pai e a finalidade da
execução são determinantes.

## Investigação no Wazuh

No Dashboard, filtre o agente afetado e pesquise o Event ID `4688` para Windows
ou os eventos de coleta disponíveis para Linux. Registre os valores exibidos:

| Campo | Windows | Linux |
|---|---|---|
| Agente | `WIN-ENDPOINT` | `LINUX-ENDPOINT` |
| IP | `10.10.10.20` | `10.10.10.30` |
| Fonte | Windows Event Log | `ps aux` e `syslog`, quando aplicável |
| Processo | Event ID `4688` | fotografia periódica |
| Usuário | campo do evento | usuário exibido pelo processo |
| Regra e nível | valor observado | valor observado |

Correlacione o horário do alerta com:

- eventos de autenticação;
- comandos `sudo` ou administrativos;
- alterações detectadas pelo FIM;
- serviços iniciados ou interrompidos;
- conexões de rede, quando houver evidência registrada.

Se um campo não estiver presente no alerta, registre a ausência como limitação.

## MITRE ATT&CK

O mapeamento depende do comportamento realmente observado:

- **T1059.003 — Windows Command Shell**, quando houver execução relevante de
  `cmd.exe` e a linha de comando fizer parte do comportamento investigado;
- **T1059.004 — Unix Shell**, quando uma execução relevante de shell Unix for
  observada no Linux.

O simples aparecimento de um shell durante um teste autorizado não comprova
abuso. A técnica só deve ser registrada quando houver justificativa baseada na
linha de comando, no usuário, no processo pai e no contexto.

## Resposta recomendada

Para um processo de teste autorizado:

- registrar a finalidade;
- confirmar o encerramento do processo, quando aplicável;
- classificar como atividade esperada;
- encerrar a triagem com a evidência correspondente.

Para um processo sem autorização:

- preservar os eventos e a linha de comando;
- identificar o usuário e o processo pai;
- verificar persistência, arquivos e conexões relacionadas;
- considerar isolamento do endpoint conforme o procedimento aprovado;
- bloquear ou remover o artefato somente após preservar evidências e obter
  autorização;
- escalar o caso se houver impacto ou indícios de comprometimento.

As ações acima são recomendações, salvo quando houver registro de execução.

## Evidências e registro

Cada análise deve conter:

- ID interno da investigação;
- endpoint e IP;
- data e hora;
- processo, PID, PPID e usuário, quando disponíveis;
- caminho e linha de comando, quando disponíveis;
- processo pai;
- evento ou regra do Wazuh;
- evidências locais e no Dashboard;
- classificação e justificativa;
- limitações;
- ação executada ou recomendada.

Uma captura de tela ou alerta sem contexto não é suficiente para concluir que
um processo é malicioso.

## Limitações conhecidas

- O Event ID `4688` depende da auditoria configurada no Windows.
- O Linux utiliza coleta periódica de `ps aux`, sem garantia de capturar
  processos efêmeros.
- O módulo não possui Sysmon, auditd, EDR ou análise de memória.
- Nem todo processo possui caminho, assinatura, PPID ou linha de comando
  disponível no evento recebido.
- A ausência de alerta não prova ausência de execução.

## Referências do laboratório

- [`01-Architecture`](../01-Architecture): arquitetura, redes, IPs e portas;
- [`02-Windows-Monitoring`](../02-Windows-Monitoring): auditoria e geração do
  Event ID `4688`;
- [`03-Linux-Monitoring`](../03-Linux-Monitoring): coleta de processos,
  serviços e FIM;
- [`05-Account-and-Privilege-Changes`](../05-Account-and-Privilege-Changes):
  contexto de usuários e privilégios relacionados.
