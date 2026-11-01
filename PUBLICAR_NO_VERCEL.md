# AgendaStudio v0.10.2 — Publicação no Vercel

## Diagnóstico do 404 anterior

O repositório enviado ao GitHub continha somente `.env.example`, `.gitignore`, documentação e `package.json`.
Faltavam `src/`, `public/` e `painel/`. O Vercel publicou um projeto vazio de páginas.

Este pacote foi ajustado para o Vercel:
- `src/server.mjs`: agora **exporta** o Express e só abre porta quando executado fora do Vercel;
- `public/app/`: site de agendamento e todos os seus arquivos estáticos;
- `public/painel/`: página de login e painel;
- `vercel.json`: reescritas sem loops para `/`, `/app/` e `/painel/`, e cabeçalhos de privacidade do painel;
- rotas `/v1/*` e `/health`: processadas pela função Express (sem reescritas genéricas).

## 1. Atualize o GitHub com o projeto COMPLETO

Não envie só os cinco arquivos da raiz. No repositório `agendastudio`, devem existir pelo menos:

```
.env.example
.gitignore
package.json
vercel.json
src/
public/
supabase/
tests/
painel/
web/
```

### Jeito simples no VS Code (Git instalado)

1. Descompacte o ZIP **pronto para Vercel** numa pasta nova.
2. Abra o terminal PowerShell nessa pasta, onde estão `package.json`, `src` e `public`.
3. Rode os comandos abaixo, alterando a URL somente se seu GitHub for diferente:

```powershell
git init
git branch -M main
git remote add origin https://github.com/7klucass/agendastudio.git
git add .
git status
git commit -m "AgendaStudio pronto para Vercel"
git push -u origin main
```

Se a pasta do repositório já existir localmente, prefira **copiar o conteúdo do ZIP para essa pasta** e fazer `git add .`, `git commit` e `git push`. Se `git push` reclamar de histórico remoto diferente, não use `--force`; clone o repositório e copie o projeto para ele.

**NUNCA** envie `.env` ou qualquer chave secreta para GitHub. Só `.env.example` com valores fictícios.

## 2. Configurar o projeto no Vercel

Em **Vercel → projeto agendastudio → Settings → Build and Deployment**:

- Framework Preset: **Other** / detecção automática de Express (não Next.js, não Vite);
- Root Directory: **./** (a pasta raiz com `package.json`, `src` e `public`);
- Build Command: **deixe vazio/automático**;
- Output Directory: **deixe vazio/automático**;
- install: **npm install** (padrão);
- Node.js: **22.x**, se a opção estiver disponível.

Use **Deployments → Redeploy** após conectar ao commit atualizado, ou aguarde o deploy automático do GitHub.

## 3. Variáveis de ambiente

Em **Settings → Environment Variables**, defina para **Production**:

- `SUPABASE_URL`: URL HTTPS do seu projeto Supabase;
- `SUPABASE_PUBLISHABLE_KEY`: chave pública/anon do Supabase;
- `SUPABASE_SECRET_KEY`: **chave secret/service_role**, *somente* nas variáveis do servidor Vercel;
- `APP_NO_INDEX`: `true` durante testes;
- `NODE_ENV`: `production` (o Vercel geralmente define automaticamente).

Não é necessário `PORT`. Não envie essas chaves no chat, no código, no GitHub ou como variáveis iniciadas por `NEXT_PUBLIC_`/`VITE_`.

## 4. Checar os caminhos publicados

Com seu domínio `https://SEU-SITE.vercel.app`:

- `/` → site de agendamento;
- `/app/` → site de agendamento;
- `/painel/` → login do proprietário;
- `/health` → JSON com `ok: true`;
- `/v1/public/barbearia-aurora/catalog` → dados públicos da loja (se existir no Supabase).

Se `/` abrir mas `/health` falhar, revise as variáveis e os Function Logs no Vercel. Se aparecer 404 global, confira se as pastas e o `vercel.json` estão realmente no GitHub e se o Root Directory é a raiz correta.

## Avisos para produção

- Este projeto usa `sharp`; as fotos dos barbeiros podem esbarrar nos **limites de tamanho de requisição das Vercel Functions**. Antes de vender, valide uploads no ambiente real e considere envio direto ao Supabase Storage com permissões apropriadas.
- Rate limit do Express em memória não é global entre instâncias serverless. Antes de receber tráfego real, configure proteção compartilhada por IP/usuário (Redis/Upstash ou equivalente) e monitoramento.
- Teste permissões, isolamento entre barbearias, HTTPS, termos/privacidade preenchidos e backups antes de aceitar clientes pagantes.
- O URL do painel não é um segredo: a segurança vem do login, do JWT, do papel do proprietário e das políticas RLS do Supabase.
