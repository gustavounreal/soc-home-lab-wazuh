Windows — criação de conta (4720)

Event Viewer mostrando o evento real;
conta criada, usuário responsável e horário.# SOC Home Lab — Account and Privilege Changes

![Status](https://img.shields.io/badge/Status-Em_Desenvolvimento-yellow)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-blue)
![Scope](https://img.shields.io/badge/Scope-SOC_N1-orange)

## Sobre o cenário

Este módulo documenta a identificação e a triagem de alterações controladas em
contas locais e grupos privilegiados no SOC Home Lab. O cenário utiliza os
endpoints já definidos em
[`01-Architecture`](../01-Architecture), sem Active Directory ou novas
máquinas:

- `WIN-ENDPOINT` (`10.10.10.20`): Windows 11 Pro;
- `LINUX-ENDPOINT` (`10.10.10.30`): Ubuntu Server 24.04 LTS;
- `SOC-SERVER` (`10.10.10.10`): Wazuh Manager, Indexer e Dashboard;
- rede interna `SOC-LAB` (`10.10.10.0/24`);
- acesso administrativo ao Dashboard por
  `https://192.168.56.10`.

O objetivo é demonstrar o fluxo de um analista SOC N1: identificar quem
executou uma alteração, confirmar o evento no endpoint, avaliar se a atividade
foi autorizada, classificar o risco e registrar a resposta adequada.

## Escopo e limites

Este módulo cobre:

- criação e remoção de contas locais;
- inclusão e remoção de membros em grupos privilegiados;
- identificação do usuário responsável, quando o evento fornecer essa
  informação;
- correlação entre evento do endpoint e alerta no Wazuh;
- triagem, classificação e recomendação de resposta.

Uma conta criada ou adicionada a um grupo administrativo não é, isoladamente,
prova de elevação maliciosa de privilégio. A interpretação depende do
responsável, do horário, da autorização, do contexto e de eventos relacionados.

Este módulo não cobre Active Directory, domínio Windows, IAM corporativo,
persistência avançada ou resposta automática.

## Fluxo técnico

```text
WIN-ENDPOINT (10.10.10.20)  --+
                              |
LINUX-ENDPOINT (10.10.10.30) -+--> SOC-SERVER (10.10.10.10)
                                    Wazuh Manager / Indexer / Dashboard

Host físico --> Host-Only (192.168.56.10) --> Dashboard
```

Os agentes enviam telemetria pela rede interna `SOC-LAB` usando o canal
operacional do Wazuh em TCP `1514`. A interface Host-Only é utilizada somente
pelo host físico para acessar o Dashboard.

## Cenários demonstrados

### Windows — conta local e grupo Administrators

O script
[`02-windows-command-audit.ps1`](../02-Windows-Monitoring/scripts/02-windows-command-audit.ps1)
habilita a auditoria necessária e executa uma criação controlada de conta e
uma inclusão no grupo local `Administrators`.

Eventos esperados:

| Event ID | Atividade |
|---:|---|
| `4720` | Criação de uma conta de usuário |
| `4732` | Inclusão de membro em grupo local |

O nome da conta de teste e o resultado devem ser confirmados no endpoint antes
de serem registrados como evidência. Não reutilize credenciais reais.

### Linux — conta local e grupo sudo

No `LINUX-ENDPOINT`, as operações controladas documentadas no módulo Linux são:

```bash
sudo useradd -m -s /bin/bash labuser01
sudo usermod -aG sudo labuser01
sudo userdel -r labuser01
```

As operações devem ser executadas somente durante o teste autorizado. Confirme
o resultado localmente:

```bash
getent passwd labuser01
id labuser01
getent group sudo
```

Após a remoção, confirme que a conta de teste não permanece no sistema. Os
registros em `/var/log/auth.log` e os eventos encaminhados pelo Wazuh devem ser
correlacionados com o horário e o usuário responsável.

## Validação no Wazuh

No Dashboard, filtre o agente correspondente ao endpoint testado e consulte os
eventos no intervalo da execução. Registre somente valores observados:

| Campo | Windows | Linux |
|---|---|---|
| Agente | `WIN-ENDPOINT` | `LINUX-ENDPOINT` |
| IP do agente | `10.10.10.20` | `10.10.10.30` |
| Atividade | `4720` e `4732` | `useradd`, `usermod`, `userdel` |
| Conta de teste | valor criado no Windows | `labuser01` |
| Grupo | `Administrators` | `sudo` |
| Usuário responsável | campo do evento | usuário que executou `sudo` |
| Regra e nível | valor exibido pelo Wazuh | valor exibido pelo Wazuh |

O identificador e o nível da regra do Wazuh podem variar conforme a versão e o
ruleset instalados. Não substitua o valor observado por um valor presumido.

## Triagem N1

Para cada alteração, responda:

1. Qual endpoint foi alterado?
2. Qual conta foi criada, removida ou modificada?
3. Qual grupo recebeu a conta?
4. Quem executou a ação?
5. A ação estava autorizada e prevista?
6. O horário coincide com uma janela de manutenção ou teste?
7. Existem logons, processos ou comandos relacionados?
8. Houve acesso bem-sucedido depois da alteração?
9. A alteração foi revertida quando o teste terminou?

Classifique o achado como:

- **Atividade autorizada:** alteração planejada e confirmada;
- **Suspeita:** alteração sem contexto suficiente ou fora do padrão;
- **Incidente:** alteração não autorizada com evidências relacionadas de abuso
  ou comprometimento.

Uma alteração administrativa isolada não deve ser classificada
automaticamente como incidente.

## Investigação e evidências

Registre:

- ID interno do teste;
- data e hora inicial e final;
- endpoint e IP;
- conta afetada;
- grupo anterior e grupo posterior, quando aplicável;
- usuário responsável;
- evento ou regra do Wazuh;
- comando ou processo relacionado;
- evidência local e evidência no Dashboard;
- autorização conhecida;
- classificação e justificativa;
- limitações da análise.

No Windows, preserve os dados relevantes dos eventos `4720` e `4732`, incluindo
o sujeito que executou a ação e a conta afetada. No Linux, preserve o trecho de
`auth.log` e a saída de `id`, `getent` ou do comando correspondente.

## MITRE ATT&CK

O comportamento pode ser contextualizado, conforme a evidência:

- **T1136 — Create Account**, quando uma nova conta é criada;
- **T1098 — Account Manipulation**, quando atributos, grupos ou permissões de
  uma conta são alterados.

O mapeamento não comprova intenção maliciosa. A justificativa deve explicar
qual evento foi observado e por que a técnica é pertinente ao comportamento.

## Resposta recomendada

Para uma alteração autorizada:

- confirmar o responsável;
- registrar a finalidade;
- validar a reversão da conta ou do grupo de teste;
- encerrar a triagem como atividade esperada.

Para uma alteração não autorizada:

- preservar os eventos e horários;
- validar logons e processos relacionados;
- desabilitar ou remover a conta conforme o procedimento aprovado;
- revisar a associação a grupos privilegiados;
- redefinir credenciais se houver indício de comprometimento;
- escalar o caso para investigação quando houver evidência adicional.

As ações acima devem ser descritas como recomendadas, salvo quando houver
registro de que foram realmente executadas.

## Evidências Visuais

As evidências deste módulo estão organizadas em
[`05-Account-and-Privilege-Changes/imagens`](./imagens):

### Windows — evento 4720

![Visualizador de Eventos — criação de usuário](<./imagens/01-Visualizador de Eventos_ Criação de Usuário.png>)

A captura mostra o evento `4720` no `WIN-ENDPOINT`:

- conta criada: `soc-test01`;
- usuário responsável: `Administrador`;
- computador: `WIN-ENDPOINT`;
- IP do endpoint: `10.10.10.20`;
- data e hora observadas: `17/11/2026 16:36:28`;
- Event Record ID: `46287`;
- Wazuh Manager: `10.10.10.10` via TCP/1514.

### Linux — execução e verificação local

![Demonstração de gerenciamento de usuários Linux](<./imagens/02-Demonstração de gerenciamento de usuários Linux.png>)

A captura mostra, no `LINUX-ENDPOINT`, a criação, a inclusão no grupo
`sudo`, a verificação e a remoção de `labuser01`:

- Ubuntu Server 24.04 LTS;
- IP: `10.10.10.30`;
- `useradd`: `16:52:14`;
- associação ao grupo `sudo`: `16:52:32`;
- `userdel -r`: `16:52:46`;
- confirmação final sem a conta no sistema e no grupo `sudo`;
- janela total observada: `16:52:14` a `16:52:53`;
- Wazuh Manager: `10.10.10.10` via TCP/1514.

### Wazuh — correlação dos eventos

![Painel Wazuh de eventos correlacionados](<./imagens/03-Painel Wazuh de Eventos Correlacionados.png>)

A captura do Dashboard Wazuh apresenta cinco eventos no intervalo de
`17/11/2026 16:35:00` a `16:53:00`, envolvendo dois agentes:

| Agente | IP | Evento/regra | Atividade | Conta | Grupo | Responsável | Nível |
|---|---|---:|---|---|---|---|---:|
| `LINUX-ENDPOINT` | `10.10.10.30` | `550` | Conta removida (`userdel -r`) | `labuser01` | — | `root` | 12 |
| `LINUX-ENDPOINT` | `10.10.10.30` | `554` | Usuário adicionado ao grupo `sudo` | `labuser01` | `sudo` | `root` | 10 |
| `LINUX-ENDPOINT` | `10.10.10.30` | `553` | Conta criada (`useradd`) | `labuser01` | — | `root` | 8 |
| `WIN-ENDPOINT` | `10.10.10.20` | `4732` | Membro adicionado ao grupo privilegiado | `soc-test01` | `Administrators` | `Administrator` | 8 |
| `WIN-ENDPOINT` | `10.10.10.20` | `4720` | Conta de usuário criada | `soc-test01` | — | `Administrator` | 7 |

Todos os eventos aparecem com status `Open`. As regras e os níveis devem ser
tratados como os valores observados nesta execução; eles podem variar conforme
a versão do Wazuh e o ruleset instalado.

As três imagens representam uma atividade administrativa controlada. Elas
comprovam alterações de contas e grupos e sua correlação no Wazuh, mas não
comprovam, isoladamente, intenção maliciosa ou elevação indevida de privilégio.

## Limitações conhecidas

- Os testes utilizam contas locais; não representam um domínio corporativo.
- A disponibilidade dos campos depende da auditoria e do ruleset configurados.
- O Wazuh pode receber o evento sem todos os dados necessários para determinar
  a intenção do usuário.
- A inclusão em `Administrators` ou `sudo` pode ser legítima em uma janela de
  administração.
- Este cenário não implementa resposta automática nem bloqueio preventivo.

## Referências do laboratório

- [`01-Architecture`](../01-Architecture): arquitetura, redes, IPs e portas;
- [`02-Windows-Monitoring`](../02-Windows-Monitoring): auditoria e eventos
  Windows;
- [`03-Linux-Monitoring`](../03-Linux-Monitoring): autenticação, usuários,
  sudo e coleta Linux.
