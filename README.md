# Chá de Panela — Lista de Casamento

Site completo com backend próprio: lista de presentes, reservas por convidados
e painel administrativo para editar tudo sem mexer em código.

## Estrutura

```
cha-de-panela/
├── server/
│   ├── server.js       ← API + servidor
│   ├── package.json
│   └── db.json         ← "banco de dados" (arquivo JSON), já populado com os 44 itens da planilha
└── public/
    ├── index.html       ← site do convidado
    └── admin.html        ← painel administrativo
```

## Como rodar localmente (para testar)

```
cd server
npm install
npm start
```

Acesse:
- Site: http://localhost:3000
- Admin: http://localhost:3000/admin.html

## Como hospedar de verdade

Este é um projeto **Node.js** simples (sem banco de dados externo — os dados
ficam em `server/db.json`). Ele roda em qualquer serviço que hospede apps
Node, por exemplo:

- **Railway** ou **Render** (gratuitos para projetos pequenos): conecte o
  repositório, defina o diretório raiz como `server`, comando de start
  `npm start`, e pronto.
- **Um VPS próprio**: instale Node 18+, rode `npm install && npm start`
  dentro de `server/`, e use algo como `pm2` ou um serviço `systemd` para
  manter o processo no ar. Coloque um proxy (nginx/Caddy) na frente para
  HTTPS e domínio próprio.

⚠️ **Importante sobre persistência**: os dados ficam salvos em
`server/db.json`, no próprio disco do servidor. A maioria dos serviços
gratuitos (Railway free tier, Render free tier) **reseta o disco** a cada
novo deploy — então se você atualizar o código, os dados voltam ao que
está no `db.json` enviado. Se for usar um desses, garanta que o `db.json`
enviado já tem os itens certos, e evite fazer redeploys depois que as
reservas começarem a chegar. Um VPS próprio, ou um plano pago com "disco
persistente", não tem esse problema.

## Primeiro acesso ao admin

Na primeira vez que abrir `/admin.html`, o site vai pedir para você criar
uma senha de administrador (isso só acontece uma vez). Guarde essa senha —
é ela que protege a criação/edição/exclusão de itens.

## O que dá pra fazer no admin

- Editar os textos da página inicial (título, texto de apresentação, rodapé)
- Definir a data do evento — o site mostra contagem regressiva se for no
  futuro, ou "casados há X dias" se já tiver passado
- Criar, editar e excluir itens da lista (com múltiplos links de lojas)
- Adicionar um preço estimado por item (não é puxado automaticamente de
  nenhuma loja — isso exigiria acesso que as lojas não liberam para sites
  externos, então é um campo que você preenche manualmente se quiser)
- Ver e liberar reservas feitas por convidados

## Como funciona a reserva

Qualquer visitante pode reservar um item digitando o nome. Depois disso,
o item aparece como "reservado" para todo mundo (dado real, salvo no
servidor — não é só no navegador de quem reservou). Só quem reservou
(digitando o mesmo nome) ou o admin pode liberar a reserva de novo.
