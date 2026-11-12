# DEAD DIGITAL PAY v0.4

Gateway Bitcoin **não-custodial**: o comerciante cadastra só a **chave pública estendida** (zpub/vpub, BIP84). Cada cobrança recebe um endereço novo derivado dela, e o dinheiro cai direto na carteira do comerciante. O servidor **nunca** guarda chave privada nem seed (chaves privadas são rejeitadas).

## O que funciona
- Cobranças com endereço único (BIP84 bech32), URI BIP21, página de pagamento `/pay/:id`
- Monitor da blockchain (API Esplora/mempool.space): detecta, confirma, expira, pago a menos / a mais
- Webhooks assinados (HMAC-SHA256), com retentativas e proteção SSRF
- Painel: cobranças, carteira, API keys, webhooks, admin
- API por API key com `Idempotency-Key` atômico
- Livro-razão (`LedgerEntry`) idempotente por cobrança paga

## Instalação (testnet)
```bash
cp .env.example .env
npm run secrets            # cole JWT_SECRET, APP_ENCRYPTION_KEY, ADMIN_BOOTSTRAP_PASSWORD no .env
docker compose up -d db    # ou use seu próprio PostgreSQL e ajuste DATABASE_URL
npm install
npm run db:push            # cria as tabelas
npm run admin:create
npm run dev                # http://localhost:3000
```
Depois, no painel: **Admin → criar comerciante → Carteira (cole a vpub) → Cobranças → criar**.
Para testar sem enviar moedas: botão **Simular pgto** (só testnet, só com `ENABLE_DEV_SIMULATION=true`).
Para testar de verdade: envie tBTC de uma faucet de testnet para o endereço da cobrança.

## Obtendo a chave pública
Use uma conta **BIP84 / Native SegWit** dedicada ao gateway (ex.: Sparrow → Settings/Keystore → xpub exibido como `zpub`/`vpub`). Confira se o 1º endereço mostrado após salvar bate com o da sua carteira. Se sua carteira usa gap limit de 20, aumente-o (cada cobrança usa um endereço novo).

## API
```
POST /api/v1/invoices          X-API-Key, Idempotency-Key  { amountSats, description, merchantReference?, expiresInMinutes? }
GET  /api/v1/invoices/:id      X-API-Key
```
Webhook: cabeçalho `X-DDP-Signature: t=<unix>,v1=<hex>` onde `v1 = HMAC_SHA256(secret, "<t>.<corpo bruto>")`. Rejeite se `t` tiver mais de 5 min. Responda 2xx para confirmar.

## Mainnet
Bloqueada por padrão. Só habilite (`ALLOW_MAINNET=true`, `NODE_ENV=production`, `ENABLE_DEV_SIMULATION=false`) depois de testar tudo na testnet e de revisar `docs/PRODUCTION_READINESS.md`.

## Testes
`npm test` — vetores oficiais BIP84, regras de status, SSRF e cifra. (O servidor em si precisa de `npm install` + PostgreSQL.)
