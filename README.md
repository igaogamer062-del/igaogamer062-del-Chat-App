# Smart Chat — Sistema de Chamadas

Este pacote evolui o Chat Checklist original para um sistema de atendimento completo,
com dois fluxos no bot (**Checklist** e **Monitoramento**), roteamento por Base/Transportadora,
painel do operador com fila em tempo real, Dashboard e gestão de acessos (com papel Coordenador).

## Estrutura

- `driver-app/` — PWA do condutor. Bot pergunta Checklist ou Monitoramento, depois
  Nome/Placa/Tecnologia, e encaminha automaticamente.
- `painel/` — **novo**: painel do operador/coordenador/administrador (login, fila de
  atendimentos com chat, Dashboard, Bases/Transportadoras, Usuários/Acessos).
- `supabase/` — migrations SQL, na ordem em que devem ser executadas (veja `DEPLOY.md`).
  - `000_core_setup.sql` — **novo**: autenticação, perfis e sistema de permissões (base que faltava no pacote original).
  - `012` a `014` — Chat Checklist (mantidos como no pacote original).
  - `015_bases_monitoramento_dashboard.sql` — **novo**: Bases, Transportadoras, fluxo de
    Monitoramento (com a "planilha" de teste) e as métricas do Dashboard.
- `js/`, `edge-functions/` — mantidos do pacote original (upload de mídia do checklist).
- `DEPLOY.md` — passo a passo completo para colocar no ar (Supabase + GitHub Pages).

## Como o roteamento funciona

**Checklist:** qualquer operador com a permissão `checklist_chat` pode receber, sorteado
entre os com menos atendimentos abertos no momento (aleatório em caso de empate).

**Monitoramento:** o bot pega Placa → consulta a "planilha" de teste (`mock_fleet_drivers`,
que simula o sistema externo da transportadora) → acha a Transportadora → acha a Base
vinculada a ela (aba **Bases** do painel) → sorteia um operador vinculado àquela Base com
permissão `monitoring_chat`. Se qualquer etapa falhar (placa não achada, transportadora sem
base, base sem operador disponível), o atendimento não fica perdido: ele aparece em
**Atendimentos → Não roteados** para um Coordenador/Administrador assumir manualmente.

## Papéis

- **Operador / Líder / Supervisor** — atendem chamados (checklist e/ou monitoramento,
  conforme a permissão liberada).
- **Coordenador** — além de atender, pode vincular operadores às Bases sob sua gestão
  ("vincular operações") e ver o Dashboard filtrado só pelas suas Bases.
- **Gerente / Administrador** — acesso total: cria Bases/Transportadoras, define quem é
  Coordenador de qual Base, muda função/acesso de qualquer usuário.

## O que é MVP e o que fica para uma próxima etapa

Entregue e funcional: autenticação real, permissões por papel, roteamento automático,
fila com chat em tempo real (polling a cada 4s), encerramento de atendimento, Dashboard
com tempo médio/operadores ativos/volume por base e operador, CRUD de Bases/Transportadoras/
vínculos, gestão de usuários e acessos.

Ainda não incluído (próxima etapa, fora do escopo pedido agora): chamada de voz/áudio real
(hoje "chamada" = atendimento por chat cronometrado, como combinado), integração com a API
real da transportadora (a planilha de teste foi desenhada para ser trocada por ela sem
mexer no resto do roteamento — veja o comentário no fim de `015...sql`), polimento visual
do painel (hoje é funcional, mas simples) e notificações push para o painel do operador.
