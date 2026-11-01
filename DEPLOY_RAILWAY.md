# Publicar AgendaStudio v0.10.2 no Railway — passo a passo

**Objetivo:** publicar o site e a API num mesmo endereço HTTPS com banco e autenticação já existentes no Supabase. Este pacote está pronto para importação, **mas a hospedagem ainda não foi criada**. A conexão/autorização da conta Railway depende do proprietário da conta.

## 0. Pré-requisitos e cuidados

- Use este ZIP de publicação, não um dos ZIPs anteriores. Ele **não contém** `.env`, senhas nem chaves privadas.
- Não apague nem recrie o projeto Supabase atual. Não execute novamente as migrações SQL antigas.
- Mantenha o repositório GitHub **privado** no início.
- A chave `SUPABASE_SECRET_KEY` é **estritamente secreta**. Nunca a coloque em JavaScript público, GitHub, print, chat ou arquivo acessível pela internet.
- Para vender a clientes reais, é preciso substituir os modelos de política/termos pelos documentos próprios, revisar a LGPD e verificar licenças de imagens e avaliações.

## 1. Criar um repositório privado no GitHub

1. Entre em https://github.com/new e escolha **Private**.
2. Nome sugerido: `agendastudio`.
3. Crie o repositório vazio, sem inicializar README ou .gitignore.
4. Extraia este ZIP e **envie os arquivos de dentro** da pasta ao repositório usando *Add file > Upload files* (ou `git`), preservando as subpastas `src`, `web`, `painel`, `supabase`, `tests` e `scripts`.
5. Antes de confirmar, confira que o `package.json` está na **raiz do repositório** e que **não há nenhum arquivo `.env`**.

> Se preferir usar a integração Railway dentro do ChatGPT, conecte sua conta Railway primeiro. Sem a conexão, não é possível criar serviços na sua conta automaticamente por aqui.

## 2. Criar serviço Node.js no Railway

1. Abra https://railway.com/ e entre com sua conta.
2. Clique em **New Project > Deploy from GitHub repo**; autorize acesso ao repositório privado do AgendaStudio.
3. Escolha o repositório. O Railway detecta `package.json` e o script `npm start`.
4. Em **Variables** insira as variáveis descritas abaixo (somente por essa tela).
5. Em **Settings / Deploy** use `npm start` como comando de inicialização, caso não seja reconhecido automaticamente.
6. Configure **Healthcheck Path** = `/health` (no dashboard). O Railway define o `PORT` automaticamente.
7. Faça o deploy e aguarde o serviço constar como **Active**.
8. Em **Settings > Networking > Generate Domain**, gere seu endereço HTTPS `https://...up.railway.app`.

## 3. Variáveis de ambiente

Copie **os valores reais do seu `.env` local** para o painel **Variables** do Railway (não envie o arquivo ao GitHub):

| Nome | Valor |
|---|---|
| `SUPABASE_URL` | `https://SEU-PROJETO.supabase.co` |
| `SUPABASE_PUBLISHABLE_KEY` | `sb_publishable_...` ou a chave pública atualmente usada e funcional |
| `SUPABASE_SECRET_KEY` | `sb_secret_...` **somente em Variables do Railway** |
| `NODE_ENV` | `production` |
| `APP_NO_INDEX` | `true` durante demonstração; mude para `false` só quando for lançar publicamente |
| `TRUST_PROXY` | `true` |

**Não crie a variável `PORT` manualmente** no Railway: a plataforma fornece a porta do serviço automaticamente. `CORS_ORIGINS` pode ficar vazio quando site e API estão no mesmo domínio. Caso use outros front-ends, informe exatamente os domínios HTTPS permitidos, separados por vírgulas.

## 4. Testar os três endereços

Depois de obter `https://SEU-ENDERECO.up.railway.app`, teste no navegador:

- `https://SEU-ENDERECO.up.railway.app/health` — deve retornar JSON com `"ok":true`.
- `https://SEU-ENDERECO.up.railway.app/app/` — página de agendamento do cliente.
- `https://SEU-ENDERECO.up.railway.app/painel/` — login exclusivo do proprietário.

No PowerShell, execute (adaptando sua URL):

```powershell
$env:BASE_URL = "https://SEU-ENDERECO.up.railway.app"
npm run smoke
```

Se o catálogo falhar mas as três páginas abrirem, confira `SUPABASE_URL`, `SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_SECRET_KEY` e as migrações aplicadas no Supabase. O script não precisa nem pede credenciais de usuário.

## 5. Configurar Supabase Auth

No Supabase, abra **Authentication > URL Configuration**. Atualize **Site URL** para o endereço HTTPS oficial. Se usar login por link, confirmação de e-mail ou redefinição de senha, inclua os caminhos específicos do domínio em **Redirect URLs**. O login por e-mail e senha atual utiliza Supabase Auth diretamente e não necessita criar novos usuários.

## 6. Teste de ponta a ponta antes de divulgar

- Conferir funcionamento no computador e no celular.
- Criar um agendamento de teste, verificar bloqueio de choque de horário e conferir exibição no painel.
- Confirmar e cancelar a reserva, validar os filtros por mês/data/profissional.
- Testar exclusão/desativação de serviço e profissional em dados fictícios.
- Testar acesso negado a uma conta que não pertence à barbearia (isolamento por tenant/RLS).
- Confirmar que as fotos funcionam e que a chave secreta jamais aparece em `/app`, `/painel` ou `/v1/public/config`.
- Personalizar os documentos legais e revisar os consentimentos. O template não constitui, por si só, conformidade com a LGPD.
- Estabelecer backups, controle de custos, alertas e rotina de suporte.

## 7. Domínio próprio (depois que o subdomínio funcionar)

Se comprar um domínio, em Railway **Settings > Networking > Custom Domain** configure o domínio. O Railway mostra quais registros DNS devem ser inseridos no seu registrador. Não adivinhe registros DNS. Atualize então a **Site URL** no Supabase para o domínio definitivo.

## Diagnóstico rápido

- **Build failed:** confira se `package.json` está na raiz e se o Node.js 22 foi selecionado; verifique os logs do Railway.
- **Application failed to respond:** confira Variables, logs e se `npm start` está ativo. Não force `PORT=3342`; a porta de produção é do Railway.
- **Erro no catálogo:** verifique as chaves e o projeto do Supabase. Nunca cole as chaves no chat.
- **Painel não abre:** confira o caminho com barra final `/painel/`, o status em `/health` e os logs do Railway.
- **Login não tem barbearia:** confira a vinculação do usuário a `shop_members` no Supabase, sem inserir contas novas às cegas.

Links: [Deploy do Railway](https://docs.railway.com/quick-start) · [Variáveis](https://docs.railway.com/variables) · [Domínios](https://docs.railway.com/platform/compare-to-vercel) · [URLs do Supabase Auth](https://supabase.com/docs/guides/auth/redirect-urls).
