# SOC Home Lab — Incident Investigation

![Status](https://img.shields.io/badge/Status-Em_Desenvolvimento-yellow)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-blue)
![Scope](https://img.shields.io/badge/Scope-SOC_N1-orange)

## Sobre o módulo

Este módulo define a metodologia de investigação dos alertas gerados nos
cenários do SOC Home Lab. Ele não cria uma nova fonte de eventos: organiza e
correlaciona as evidências coletadas pelos módulos de monitoramento e detecção.

O objetivo é demonstrar que um analista SOC N1 consegue responder, com
evidências:

- o que aconteceu;
- quando aconteceu;
- qual ativo foi afetado;
- qual usuário e origem estavam envolvidos;
- quais eventos estão relacionados;
- se a atividade foi autorizada, suspeita ou um incidente;
- o que foi possível determinar e quais são as limitações.

## Ambiente investigado

| Componente | IP interno | Papel |
|---|---:|---|
| `SOC-SERVER` | `10.10.10.10` | Wazuh Manager, Indexer e Dashboard |
| `WIN-ENDPOINT` | `10.10.10.20` | Endpoint Windows monitorado |
| `LINUX-ENDPOINT` | `10.10.10.30` | Endpoint Linux monitorado |
| `KALI-ATTACKER` | `10.10.10.40` | Origem de testes autorizados |

Os endpoints e o Manager comunicam-se pela rede interna `SOC-LAB`
(`10.10.10.0/24`). O host físico acessa o Dashboard pela interface Host-Only
em `https://192.168.56.10`; essa rede não é utilizada como caminho de
telemetria dos endpoints.

## Fontes de evidência

Utilize as fontes abaixo conforme o cenário:

| Fonte | Uso na investigação |
|---|---|
| Wazuh Dashboard | alerta, regra, nível, agente, horário e campos normalizados |
| `/var/log/auth.log` | SSH, falhas de autenticação, `sudo` e contas Linux |
| `/var/log/syslog` | serviços e atividades do sistema Linux |
| FIM do Wazuh | criação e alteração de arquivos em `/etc` e `/opt/lab-fim` |
| Windows Event Log | logon, contas, grupos e criação de processos |
| `ps aux` | fotografia periódica dos processos Linux |
| Scripts de teste | atividade gerada e horário esperado |

Uma fonte deve confirmar a outra sempre que possível. Um alerta isolado ou
uma captura sem contexto não é suficiente para concluir que houve incidente.

## Casos investigáveis

### Brute force SSH

Use o módulo
[`04-Brute-Force-Detection`](../04-Brute-Force-Detection) para investigar
falhas repetidas originadas pelo `KALI-ATTACKER` (`10.10.10.40`) contra o
`LINUX-ENDPOINT` (`10.10.10.30`).

Confirme:

- linhas `Failed password` no `auth.log`;
- quantidade e intervalo das tentativas;
- usuário e porta de destino;
- eventual sucesso posterior;
- correspondência entre o horário local e o Dashboard;
- se a origem era um teste autorizado.

Uma sequência gerada pelo próprio laboratório deve ser classificada como
atividade autorizada, mesmo que o padrão seja semelhante a brute force.

### Alterações de contas e privilégios

Use o módulo
[`05-Account-and-Privilege-Changes`](../05-Account-and-Privilege-Changes)
para investigar:

- Windows Event IDs `4720` e `4732`;
- Linux `useradd`, `usermod`, `userdel` e grupo `sudo`;
- usuário responsável e autorização;
- estado da conta antes e depois da alteração.

Criação de conta ou inclusão em grupo privilegiado não prova, isoladamente,
elevação maliciosa. A conclusão exige contexto e eventos relacionados.

### Processos suspeitos

Use o módulo
[`06-Suspicious-Processes`](../06-Suspicious-Processes) para correlacionar:

- Windows Event ID `4688`;
- nome, caminho, usuário, PID, PPID e linha de comando, quando disponíveis;
- processos Linux observados pela coleta periódica `ps aux`;
- autenticação, `sudo`, serviços e FIM próximos ao horário do processo.

No Linux, a ausência de um processo na coleta não prova que ele não foi
executado, pois `ps aux` é uma fotografia periódica.

## Procedimento de investigação N1

### 1. Validar o alerta

Registre o identificador do alerta, data e hora, agente, regra, nível e
descrição exatamente como exibidos no Wazuh. Separe a severidade configurada
na ferramenta da prioridade atribuída durante a triagem.

### 2. Confirmar o ativo

Compare o nome e o IP do agente com a arquitetura:

- `WIN-ENDPOINT` — `10.10.10.20`;
- `LINUX-ENDPOINT` — `10.10.10.30`;
- `SOC-SERVER` — `10.10.10.10`.

Um alerta atribuído ao agente errado deve ser tratado como problema de
contexto ou de documentação até ser validado.

### 3. Construir a linha do tempo

Organize os eventos em ordem cronológica:

| Horário | Ativo | Usuário | Origem | Evento | Fonte |
|---|---|---|---|---|---|
| valor observado | valor observado | valor observado | valor observado | descrição | fonte |

Inclua pelo menos o primeiro evento, os eventos relacionados e o último
evento observado. Não preencha lacunas com horários estimados.

### 4. Correlacionar evidências

Compare o alerta com:

- log original do endpoint;
- eventos de autenticação;
- comandos administrativos;
- criação ou alteração de arquivo;
- processos e serviços;
- origem, destino, porta e protocolo;
- atividade autorizada registrada no teste.

### 5. Classificar

Use uma classificação que possa ser justificada:

- **Falso positivo:** o alerta não representa a condição investigada;
- **Atividade autorizada:** o evento é real, mas foi gerado por teste,
  manutenção ou administração aprovada;
- **Suspeita:** há anomalia, mas a evidência ainda é insuficiente;
- **Incidente:** há atividade não autorizada e evidência suficiente de impacto
  ou comprometimento.

### 6. Registrar limitações

Documente campos ausentes, atraso de coleta, relógios divergentes, regras não
confirmadas, processos efêmeros e qualquer informação que não pôde ser
validada.

## Networking e análise de conexões

Para cada conexão relacionada ao alerta, registre:

- IP de origem;
- IP de destino;
- porta de origem, quando disponível;
- porta de destino;
- protocolo;
- direção da conexão;
- ativo que gerou o registro;
- fonte que comprovou a informação.

No ambiente atual:

- SSH de teste: `10.10.10.40` → `10.10.10.30:22/TCP`;
- telemetria do agente: endpoint → `10.10.10.10:1514/TCP`;
- acesso administrativo ao Dashboard: host físico →
  `192.168.56.10:443/TCP`, conforme configuração do servidor.

Não trate um IP interno do laboratório como indicador malicioso sem contexto.
O `10.10.10.40` é a origem esperada dos testes do Kali, não uma ameaça por si
mesmo.

DNS e HTTP/HTTPS só devem ser incluídos na investigação quando existirem
registros correspondentes. A arquitetura não fornece, por si só, evidência de
um domínio ou conexão externa específica.

## IOC Analysis

Um indicador deve ser registrado com contexto:

| Tipo | Valor | Fonte | Data da consulta | Resultado | Limitação |
|---|---|---|---|---|---|
| IP | valor observado | log ou alerta | data real | contexto | não concluir reputação sem consulta |
| Hash | valor observado | FIM ou arquivo | data real | resultado | hash antes/depois pode ser apenas alteração |
| Domínio | valor observado | log disponível | data real | resultado | não inventar domínio |

Regras para análise:

1. preserve o valor exatamente como foi observado;
2. registre a fonte e a data da consulta;
3. diferencie indicador observado de indicador malicioso confirmado;
4. documente falsos positivos e limitações da fonte;
5. não use IPs, hashes ou domínios inventados;
6. não apresente `10.10.10.40` ou `192.168.56.10` como ameaça real.

Os hashes exibidos pelo FIM devem ser copiados do evento real. Se o evento não
contiver hash, registre que o indicador não estava disponível.

## MITRE ATT&CK

O mapeamento deve ser feito somente após a investigação:

- `T1110` ou `T1110.001` para tentativas de senha, quando aplicável;
- `T1136` para criação de conta;
- `T1098` para manipulação de conta;
- `T1059.003` ou `T1059.004` para execução de shell, quando a evidência
  justificar.

Cada técnica precisa de uma justificativa baseada em evento, comando, usuário,
origem e contexto. O mapeamento não substitui a análise.

## Modelo de registro

```text
ID:
Data e hora:
Origem do alerta:
Ativo afetado:
IP do ativo:
Usuário:
Origem e destino:
Porta e protocolo:
Descrição:
Evidências:
Linha do tempo:
Classificação:
MITRE ATT&CK:
Impacto potencial:
Ação executada ou recomendada:
Limitações:
Status:
Justificativa de encerramento:
```

## Resposta e escalonamento

Uma investigação N1 deve recomendar ações proporcionais:

- atividade autorizada: documentar e encerrar;
- suspeita: preservar evidências, buscar contexto e acompanhar;
- incidente: preservar evidências, conter conforme autorização e escalar;
- dados insuficientes: registrar a lacuna e solicitar a informação necessária.

Não descreva bloqueio, isolamento, remoção de conta ou alteração de senha como
executados quando foram apenas recomendados.

## Limitações conhecidas

- O laboratório não possui uma plataforma ITSM dedicada.
- A análise depende dos campos efetivamente coletados pelo Wazuh.
- `ps aux` não captura necessariamente processos efêmeros.
- A ausência de um alerta não prova ausência de atividade.
- A classificação depende do contexto de autorização e dos eventos disponíveis.
- Este módulo documenta investigação N1; não substitui forense completa.

## Referências do laboratório

- [`01-Architecture`](../01-Architecture): topologia, IPs, redes e portas;
- [`02-Windows-Monitoring`](../02-Windows-Monitoring): eventos Windows;
- [`03-Linux-Monitoring`](../03-Linux-Monitoring): logs e FIM Linux;
- [`04-Brute-Force-Detection`](../04-Brute-Force-Detection): falhas SSH;
- [`05-Account-and-Privilege-Changes`](../05-Account-and-Privilege-Changes):
  contas e grupos;
- [`06-Suspicious-Processes`](../06-Suspicious-Processes): análise de
  processos.
