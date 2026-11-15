# Integração Google Calendar — Elisângela (preparação)

**Status em 15/11/2026:** especificação preparada. Ainda **NÃO autorizada**, **NÃO implantada**, **NÃO sincronizada**. Agenda pública permanece em demonstração.

## Contas e projetos (não misturar)

- Site: https://elisangelamassoterapeuta.com.br/agenda/
- GitHub (código): `dazarafa-cmyk/Massoterapia`, branch `main`
- Firebase/Google Cloud do mantenedor: `massoterapeuta-e97c3`
- Calendar profissional **da Elisângela**: `elisangela.massoterapeutarj@gmail.com`
- Fuso: `America/Sao_Paulo`
- Conta pessoal do mantenedor **não deve ser usada** para Calendar.
- Evento de teste já criado na agenda profissional: 26/11/2026, 09:30–10:30.

## Situação de infraestrutura observada

- `.firebaserc` aponta para `massoterapeuta-e97c3`.
- `firebase.json` usa Hosting com `public: "."` e ignora arquivos ocultos.
- Workflow `.github/workflows/firebase-hosting.yml` publica somente **Hosting**, via action-hosting-deploy.
- Logo, **não existe deploy de Cloud Functions configurado**; apenas alterar HTML não dá acesso seguro ao Google Calendar.
- Esta pasta `.integration/` é oculta e não deve ser publicada no Hosting. Nenhum token, segredo, arquivo OAuth ou dado de clientes deve entrar no GitHub, mesmo se ignorado pelo Hosting.

## Autorização necessária (ainda pendente)

1. No projeto Google Cloud `massoterapeuta-e97c3` do mantenedor, habilitar **Google Calendar API**.
2. Configurar tela de consentimento OAuth (público-alvo Externo, conforme políticas Google), e cadastrar Elisângela como usuária de teste enquanto aplicável.
3. Criar cliente OAuth **aplicativo Web**, com redirect URI HTTPS exata do backend que será efetivamente implantado. Não inventar URI antes do deploy do endpoint.
4. Implementar fluxo OAuth no backend com `state` aleatório validado e `access_type=offline`; no navegador de consentimento escolher **exclusivamente a conta da Elisângela**.
5. Persistir refresh token e client secret somente em Secret Manager/segredos do backend. Nunca enviar ao frontend ou GitHub. Restringir endpoint de início de autorização a operador autorizado.
6. Validar via API que o usuário autorizado e o calendário-alvo são `elisangela.massoterapeutarj@gmail.com`, antes de aceitar qualquer atualização. Se houver mismatch: abortar sem gravar.

**Antes do deploy**: verificar se o projeto permite Cloud Functions e o efeito de eventual ativação do plano Blaze/cobrança; não ativar sem decisão explícita do proprietário. Credenciais OAuth em modo "Testing" podem ter refresh tokens de curta duração; revisar requisitos de produção antes do uso real.

## Regras de negócio preservadas

- Mostrar somente dois meses por vez; navegar em pares deslocados um mês de novembro/2026 até novembro–dezembro/2027.
- Preservar horários e duração já definidos no HTML (incluindo deslocamentos), não criar novos slots.
- Cliente informa residencial/hotelaria, endereço e dados de contato. Dados pessoais apenas na área administrativa e no evento privado; no site público mostrar **somente disponibilidade**, nunca nome/endereço.
- Evento manual Google Calendar "ocupado" bloqueia slots com sobreposição, mesmo que não tenha sido criado pelo site.
- Evento de dia inteiro que bloqueie disponibilidade também deve ser considerado; eventos marcados como "livre" não bloqueiam.
- Reservas de site exigem operação transacional/lock no banco (Firestore) para evitar dois clientes reservarem o mesmo slot; reler Calendar antes de aceitar e tratar conflitos de atualização.
- Solicitação: bloquear imediatamente e registrar estado `pending`. Notificação de e-mail para Elisângela com aprovação/recusa por links **de uso único, expirados e validados por backend**; ação confirmada via página protegida. Aprovar => `confirmed`, bloquear. Recusar => `declined` e liberar se não houver outros impedimentos.
- Evitar liberação automática de reservas pendentes sem política aprovada.
- Cancelamento/edição manual no Calendar deve refletir na disponibilidade, com estratégia de atualização (consulta ao carregar e sincronização periódica/webhook quando configurada).
- Cliente recebe confirmação de recebimento, explicitamente **não** confirmação do atendimento.
- Sem integração ativa, manter formulário em demonstração, nunca permitir confirmação fictícia.

## Ordem técnica segura

1. Autorização e leitura **somente leitura** da agenda da Elisângela.
2. Confirmar que evento de teste 26/11/2026 09:30–10:30 torna o slot indisponível em ambiente de teste.
3. Criar backend de reservas e registros pendentes, com regras de acesso.
4. Habilitar escrita de eventos, controle de aprovação/recusa, e-mail e testes de concorrência/cancelamento.
5. Só depois publicar a experiência real mantendo rollback via GitHub.

## Checklist de validação

- [ ] Calendar API habilitada no projeto do mantenedor.
- [ ] Consentimento OAuth concluído na conta profissional da Elisângela.
- [ ] Acesso verificado sem uso de calendário pessoal.
- [ ] Leitura do evento de 26/11 09:30 validada.
- [ ] Bloqueio de slot validado em teste.
- [ ] Reservas pendentes, aprovação, recusa e notificações testadas.
- [ ] Rotinas de erro e controle de custos validados.
- [ ] Produção ativada por decisão explícita.

Este documento registra decisões e configurações; não é um sistema de reservas implantado.
