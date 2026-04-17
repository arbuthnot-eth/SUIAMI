# SUIAMI — moved

**This repo is archived.** Active development has moved to the `.SKI` monorepo:

**→ https://github.com/arbuthnot-eth/.SKI/tree/master/packages/suiami**

The `suiami` npm package is now published from that monorepo path. `npm install suiami` still gets you the latest — only the source lives elsewhere now.

## Why

We kept drifting between two sources of truth. The `.SKI` repo holds the Move contracts (`/contracts/suiami`), the browser client integration (`/src/client/suiami-*.ts`), the CCIP-read gateway (`/src/server/ens-resolver.ts`), and the SDK (`/packages/suiami`). Co-locating them removed the sync tax.

## History preserved

Tags + commit history stay browsable here as a historical record of early SUIAMI development (pre-Beldum / pre-ENS extension). The canonical Git history going forward is in the monorepo.
