# AgendaStudio — pacote para colocar online

**Base:** v0.10.2, com correção do redirecionamento `/painel/`.

## O que este pacote contém

- Site para clientes em `/app/`.
- Login e gestão do proprietário em `/painel/`.
- API Express em Node.js 22, conectada ao Supabase.
- Página `/health` para diagnóstico de deploy.
- Não indexação do painel, configuração opcional `APP_NO_INDEX=true` para fase de testes.
- Script `npm run smoke` para checar as rotas do servidor publicado.
- Guia `DEPLOY_RAILWAY.md` com instruções completas de GitHub, Railway, Supabase e segurança.

## Antes de começar

O site não está hospedado apenas por baixar este ZIP. Você precisa autorizar a hospedagem na sua conta Railway, importar este projeto no GitHub e informar suas variáveis de ambiente no Railway. **Nunca envie o `.env` ou a chave secreta para o GitHub.**

## Teste de código

```powershell
npm install
npm run check
npm test
```

## Por que as avaliações de exemplo foram retiradas?

A versão anterior tinha frases fictícias de avaliações. Para testes externos ou uso comercial, o site agora apresenta apenas um espaço identificado como preparado para depoimentos reais, sem avaliações inventadas.

## Atenção antes de captar clientes reais

Os modelos de política de privacidade e termos têm campos entre colchetes que precisam ser preenchidos e validados. Configure backups e certifique-se de que a separação entre lojas funciona de ponta a ponta.
