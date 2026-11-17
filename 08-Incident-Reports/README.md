# SOC Home Lab — Incident Reports

![Status](https://img.shields.io/badge/Status-Em_Desenvolvimento-yellow)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-blue)
![Scope](https://img.shields.io/badge/Scope-SOC_N1-orange)

## Sobre o módulo

Este módulo define o formato de registro das triagens e investigações
realizadas no SOC Home Lab. O objetivo é transformar alertas e evidências
técnicas em um registro claro, reproduzível e adequado para escalonamento.

O laboratório não possui uma plataforma ITSM dedicada. Os registros deste
módulo podem ser mantidos em Markdown, desde que preservem o histórico, as
evidências e a justificativa de cada decisão.

## Ambiente relacionado

| Componente | Endereço | Função |
|---|---:|---|
| `SOC-SERVER` | `10.10.10.10` | Wazuh Manager, Indexer e Dashboard |
| `WIN-ENDPOINT` | `10.10.10.20` | Endpoint Windows monitorado |
| `LINUX-ENDPOINT` | `10.10.10.30` | Endpoint Linux monitorado |
| `KALI-ATTACKER` | `10.10.10.40` | Origem de testes autorizados |
| Dashboard | `https://192.168.56.10` | Acesso pelo host físico |

Os registros devem respeitar a arquitetura de
[`01-Architecture`](../01-Architecture). IPs internos do laboratório não
devem ser apresentados como indicadores maliciosos sem contexto.

## Quando criar um registro

Crie um registro quando houver:

- alerta relevante no Wazuh;
- sequência de falhas de autenticação;
- alteração de conta ou grupo privilegiado;
- processo com contexto anômalo;
- alteração de arquivo relacionada a uma investigação;
- atividade que precise ser acompanhada ou escalada.

Um teste autorizado também pode ser registrado, mas deve ser classificado como
atividade esperada e identificado como teste controlado.

## Ciclo do registro

```text
Alerta recebido
      |
      v
Validação do evento e do ativo
      |
      v
Coleta e preservação das evidências
      |
      v
Triagem e classificação
      |
      +--> Atividade autorizada --> Encerramento justificado
      |
      +--> Suspeita/incidente --> Escalonamento e resposta recomendada
```

## Classificação e prioridade

Separe a severidade exibida pelo Wazuh da prioridade analítica do registro.

| Classificação | Uso |
|---|---|
| Falso positivo | O alerta não representa a condição investigada |
| Atividade autorizada | Evento real gerado por teste, manutenção ou administração aprovada |
| Suspeita | Anomalia que exige contexto adicional |
| Incidente | Atividade não autorizada com evidência suficiente de impacto ou comprometimento |

Uma sugestão simples de prioridade:

- **Baixa:** atividade esperada ou sem impacto observável;
- **Média:** atividade anômala sem confirmação de comprometimento;
- **Alta:** atividade não autorizada com risco relevante ou acesso confirmado;
- **Crítica:** impacto ativo, comprometimento confirmado ou risco imediato.

A prioridade deve sempre conter uma justificativa. Um nível alto configurado no
Wazuh não transforma automaticamente o evento em incidente crítico.

## Fontes de evidência

Registre a origem de cada informação:

- alerta e campos do Wazuh Dashboard;
- `/var/log/auth.log`;
- `/var/log/syslog`;
- Windows Event Log;
- saída de `ps`, `pstree` e `systemctl`;
- eventos do FIM;
- script utilizado para gerar o teste;
- captura de tela ou arquivo de log.

Preserve data, horário, agente, IP, usuário, regra e mensagem original quando
estiverem disponíveis. Não altere a evidência para preencher campos ausentes.

## Modelo de ticket N1

Copie o modelo abaixo para cada ocorrência:

```text
ID do registro:
Data de abertura:
Analista:
Origem do alerta:
Status: Novo | Em triagem | Escalado | Contido | Encerrado

Título:
Ativo afetado:
IP do ativo:
Usuário:
IP de origem:
IP de destino:
Porta/protocolo:
Data e hora do evento:

Descrição objetiva:

Evidências coletadas:
1.
2.
3.

Linha do tempo:
- [data/hora] evento
- [data/hora] evento

Regra e severidade no Wazuh:
Prioridade analítica:
Classificação:
MITRE ATT&CK:

Análise:

Impacto potencial:

Ação executada:
Nenhuma | descrever somente o que foi realmente executado

Ação recomendada:

Escalonamento:
Não aplicável | descrever equipe ou responsável

Limitações:

Justificativa de encerramento:
```

## Modelo de relatório técnico

Para ocorrências que exigem mais contexto, utilize:

```text
# Relatório de Incidente — <título>

## Identificação
- ID:
- Data:
- Analista:
- Status:
- Classificação:
- Prioridade:

## Resumo executivo
Descreva em poucas linhas o que foi observado e o estado atual.

## Ativo e contexto
- Ativo:
- Hostname:
- IP:
- Usuário:
- Origem:
- Destino:
- Janela analisada:

## Linha do tempo
| Data/hora | Fonte | Evento | Interpretação |
|---|---|---|---|
| | | | |

## Evidências
Liste o caminho, alerta, evento, captura ou log utilizado.

## Análise técnica
Explique a relação entre os eventos e o motivo da classificação.

## IOCs
Registre somente indicadores realmente observados, com fonte e data da consulta.

## MITRE ATT&CK
Técnica, subtécnica e justificativa baseada na evidência.

## Impacto e escopo
Descreva o que foi confirmado e o que não foi confirmado.

## Resposta
- Ações executadas:
- Ações recomendadas:
- Necessidade de escalonamento:

## Limitações
Registre dados ausentes, atrasos, escopo do teste e incertezas.

## Encerramento
Explique por que o registro foi encerrado ou por que permanece aberto.
```

## Aplicação aos cenários do laboratório

### Brute force SSH

Referencie
[`04-Brute-Force-Detection`](../04-Brute-Force-Detection) e registre:

- falhas em `/var/log/auth.log`;
- `LINUX-ENDPOINT` (`10.10.10.30`);
- origem `10.10.10.40` quando confirmada;
- usuário e janela de tentativas;
- eventual sucesso posterior;
- confirmação de que o Kali foi usado em teste autorizado.

Um teste gerado pelo laboratório deve ser encerrado como atividade autorizada,
mesmo que o padrão seja semelhante a brute force.

### Alterações de contas e privilégios

Referencie
[`05-Account-and-Privilege-Changes`](../05-Account-and-Privilege-Changes) e
registre:

- Windows Event IDs `4720` e `4732`, quando aplicáveis;
- `useradd`, `usermod` e `userdel` no Linux;
- conta afetada e grupo;
- usuário responsável;
- autorização e reversão da alteração.

Não descreva a criação de conta ou inclusão em `Administrators`/`sudo` como
elevação maliciosa sem evidência adicional.

### Processos suspeitos

Referencie [`06-Suspicious-Processes`](../06-Suspicious-Processes) e registre:

- Windows Event ID `4688`;
- nome, caminho, usuário, PID, PPID e linha de comando, quando disponíveis;
- processo observado no Linux pela coleta periódica;
- relação com autenticação, `sudo`, serviços e FIM.

Se o processo foi criado pelo script de teste, registre explicitamente a
finalidade autorizada.

## IOC Analysis no relatório

Para cada IOC, registre:

| Tipo | Valor observado | Fonte | Data da consulta | Resultado | Limitação |
|---|---|---|---|---|---|
| IP | valor real | alerta/log | data real | contexto | reputação não confirmada |
| Hash | valor real | FIM/arquivo | data real | resultado | somente se disponível |
| Domínio | valor real | log | data real | resultado | somente se observado |

Não invente indicadores. `10.10.10.40` é a origem esperada dos testes do Kali e
`192.168.56.10` é o endereço Host-Only do Dashboard; nenhum deles é malicioso
por definição.

## Evidência e cadeia de registro

Para cada arquivo ou captura, anote:

- nome e caminho;
- data de coleta;
- origem;
- descrição;
- relação com o alerta;
- integridade, se um hash tiver sido realmente calculado.

Não declare que um hash foi calculado quando ele não estiver disponível. A
evidência original deve permanecer inalterada.

## Encerramento e escalonamento

Encerre como **atividade autorizada** quando o teste, responsável e resultado
estiverem confirmados.

Mantenha como **suspeita** quando faltarem contexto ou autorização.

Escale como **incidente** quando houver atividade não autorizada, acesso
confirmado, impacto, persistência ou evidências relacionadas suficientes.

O registro deve diferenciar:

- ação executada;
- ação recomendada;
- ação aguardando aprovação;
- informação que ainda precisa ser coletada.

## Limitações conhecidas

- Markdown não substitui uma plataforma ITSM em ambiente corporativo;
- o relatório depende dos campos efetivamente coletados pelo Wazuh;
- a ausência de um campo não deve ser preenchida por suposição;
- o laboratório não implementa resposta automática;
- este módulo documenta registro e escalonamento N1, não investigação forense
  completa.

## Referências do laboratório

- [`01-Architecture`](../01-Architecture): arquitetura e topologia;
- [`02-Windows-Monitoring`](../02-Windows-Monitoring): eventos Windows;
- [`03-Linux-Monitoring`](../03-Linux-Monitoring): eventos Linux;
- [`04-Brute-Force-Detection`](../04-Brute-Force-Detection): falhas SSH;
- [`05-Account-and-Privilege-Changes`](../05-Account-and-Privilege-Changes):
  contas e privilégios;
- [`06-Suspicious-Processes`](../06-Suspicious-Processes): processos;
- [`07-Incident-Investigation`](../07-Incident-Investigation): metodologia de
  investigação.
