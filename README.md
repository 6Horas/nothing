# Nyx_aim — site (redesign)

Next.js (App Router) + Tailwind v4 + shader de fundo via `@basementstudio/shader-lab`.

## Rodando localmente

```bash
npm install
npm run dev
```

Abre em http://localhost:3000

## Estrutura

```
src/
  app/
    layout.tsx        # fontes, shader de fundo, nav, footer
    page.tsx           # Home
    projects/page.tsx
    equipment/page.tsx
    contact-me/page.tsx
  components/          # um componente por bloco visual (hero, twitch card,
                        # widgets placeholder, nav, rows...)
  content/
    types.ts            # shape de todo o conteudo do site
    site-content.ts      # ⚠️ EDITE ESTE ARQUIVO pra trocar qualquer texto
  lib/
    get-content.ts       # unico ponto de leitura de conteudo — e onde o
                         # futuro painel Admin entra (troca a fonte dos
                         # dados sem mexer em nenhuma pagina/componente)
```

## Trocar textos

Todo texto editavel (nome, bio, titulos, labels de botao, itens de
Projects/Equipment/Contact, etc.) fica em `src/content/site-content.ts`.
Edita esse arquivo, roda `npm run dev` de novo (ou o build) e pronto —
nenhum componente precisa mudar.

## Widgets com dados reais

Todos os quatro widgets abaixo ja buscam dados de verdade (sem mock).
Cada um tem uma rota de API server-side em `src/app/api/**` que guarda as
chaves/segredos no servidor — o browser nunca ve essas chaves.

1. **Copie `.env.example` pra `.env.local`** e preenche as chaves (ver
   comentarios em cada variavel). `.env.local` ja esta no `.gitignore`.
2. **Preenche os identificadores publicos** (nao sao segredo, ficam em
   `src/content/site-content.ts`): login da Twitch, IDs de canal do
   YouTube, usuario do Last.fm, identificadores dos jogos na tracker.gg.

| Widget | Fonte de dados | Onde configurar |
| --- | --- | --- |
| Twitch ao vivo | Helix API (`streams`), token via client-credentials | `TWITCH_CLIENT_ID` / `TWITCH_CLIENT_SECRET` + `twitchLive.channelLogin` |
| Ultimo video | Feed RSS publico do YouTube (sem API key) | `latestVideo.channelIds` (IDs `UC...`, um por canal) |
| Ouvindo agora | Last.fm API 2.0 (`user.getrecenttracks`, Scrobble API) | `LASTFM_API_KEY` + `nowPlayingWidget.lastfmUsername` |
| Ranks | tracker.gg API, com fallback manual | `TRACKERGG_API_KEY` + `ranksWidget.games[].trackerGgIdentifier` |

### Detalhes por widget

- **Twitch**: `TwitchLiveCard` faz polling em `/api/twitch/status` a cada
  60s. Ao vivo → "{nome} está **Ao vivo**" (verde, com bolinha piscando)
  + `Titulo da stream // Jogo`. Offline → "{nome} está off".
- **Ultimo video**: busca o RSS de cada canal em `latestVideo.channelIds`,
  pega o video mais recente entre todos, e usa o **nome do canal que veio
  do proprio feed do YouTube** (`feed.author.name`) no eyebrow
  "Ultimo video // {canal}" — nao precisa cadastrar o nome do canal na mao.
- **Now Playing**: usa a Last.fm Scrobble API 2.0. Pra funcionar, o
  Last.fm do Nyx precisa estar conectado ao Spotify (ou outro player) com
  scrobbling ativo. Mostra "Tocando agora" ou "Ultima musica ouvida".
- **Ranks**: tenta a tracker.gg primeiro; se a chave nao tiver acesso ao
  jogo (comum pra Overwatch/Deadlock — a tracker.gg libera oficialmente
  so uma lista curta de titulos, ver https://tracker.gg/developers), cai
  automaticamente pro **fallback manual** (`manualFallback.rankName` /
  `rankImageSrc` em `site-content.ts`, editado a mao sempre que o rank
  mudar).

## Rotas antigas -> novas

O site antigo (Amplify, HTML puro) usava `/Home`, `/Projects`,
`/Equipment`, `/Contact-Me`. As rotas novas sao `/`, `/projects`,
`/equipment`, `/contact-me`. Se tiver links externos apontando pras
rotas antigas, adicionar redirects no `next.config.ts` antes de trocar
o dominio de producao.

## Admin (fora do escopo por enquanto)

`src/lib/get-content.ts` ja foi desenhado pra isso: hoje `getContent()`
retorna o objeto estatico de `site-content.ts`; quando o dashboard do
Admin existir, essa funcao passa a buscar de uma API/DB e todo o resto
do site continua igual.
