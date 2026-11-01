# Contributing

Everyone is welcome, and there are three ways in. You can do any of them:

1. **Sign listings.** Claim or sign your own listing (logo, site link, and for guilds a Discord invite).
2. **Send pull requests** to this hub (listing proposals) or to the node (code).
3. **Run your own node.** For a guild, DAC, project or council. The more nodes, the better.

Governance: [PhoenixClaw01/phoenix-gate](https://github.com/PhoenixClaw01/phoenix-gate).

## Ground rules

- Text, links and images only. Nothing here moves funds, touches game keys or casts votes.
- Never commit a private key, seed phrase, key file, `.env` file or personal data. Key files stay outside every repository.
- Commit with your GitHub noreply address (`<id>+<username>@users.noreply.github.com`), not a personal mailbox.
- Logos and images: a public `https` URL you own or are allowed to use, or a file added to this repository by pull request. Always include alt text.
- Use the project's own public `https` page. No shortened links and no tracking parameters.
- A `referral` listing must carry a plain disclosure.
- No financial claims or calls to action in listing text.

## Send a listing proposal

1. Fork this repository.
2. Copy `proposals/TEMPLATE.listing.json` to `proposals/<short-id>.json` and fill it in. The fields are described in `README.md` and `VERIFY.md`. Keep `asOf` to today's UTC date and `expiresAt` no more than 90 days later.
3. Check it against the node's schema (from a clone of sa-ecosystem-node):

   ```bash
   npm ci && npm run build
   node -e 'import("./dist/hub/listing.js").then(m=>{const r=m.inspectListing(JSON.parse(require("fs").readFileSync(process.argv[1],"utf8")).listing);console.log(r.ok?"ok":m.listingFailureMessage(r.reason));process.exit(r.ok?0:1)})' /path/to/proposals/<short-id>.json
   ```

4. Open a pull request. Say who you are and how a signer can confirm the listing is yours (a wallet signature, a code on your site, or a public post).

For a guild, copy `proposals/TEMPLATE.guild.pending.json` instead and check it against the guild schema (`dist/hub/guild.js`). Rules for both are in `VERIFY.md`.

## Claim your listing (self-claim)

Already listed by us from your public links? Or a player who wants your own player listing? Claim it with your wallet. Plain-English version: `INVITE-QUICKSTART` (shared with invites). Full spec: `docs/claim-and-verify.md` in the node repository.

1. **Build the claim** (in a built clone of sa-ecosystem-node). The hub id is `https://github.com/phoenixclaw01/sa-ecosystem-hub`.

   ```bash
   # guild
   npm run hub-claim -- build --wallet <your address> --hub https://github.com/phoenixclaw01/sa-ecosystem-hub \
     --subject guild --guild-name "<guild name as listed>" --nonce <short-unique-id> \
     --expires <ISO date> --out claim.json
   # business: --subject business --listing-id <listingId>
   # player:   --subject player --listing-id <new id> --profile <your Player Profile account>
   ```

   Optional notes for the curator: `--x-handle <h> --x-proof https://x.com/<h>/status/<id>`, `--discord-note "<note>" --discord-invite https://discord.gg/<code>`, and `--show-handles` if you want your handle shown on `claim_status`.
2. **Sign** the `message.utf8` text with your wallet's sign-message (Phantom or similar). Do not sign a transaction.
3. **Attach** the signature (no key is read): `npm run hub-claim -- attach --payload claim.json --public-key <your address> --signature <sig> --out claim.signed.json`.
4. **Check it** locally: `npm run hub-claim -- verify --file claim.signed.json --hub https://github.com/phoenixclaw01/sa-ecosystem-hub`.
5. **Send it** by pull request into `proposals/`, or attach it to an issue. A hub editor records it with `hub-add-update --payload`.

When it goes active:
- **Player:** automatically, when your wallet is a current key on the Player Profile you named (one profile backs one identity).
- **Guild or business:** when curators countersign it (`sign-hub-update --approve`) up to the node's threshold. Your own signature is not an approval.

Then publish your own updates for that identity only (logo, links, site, Discord invite, description, or `withdrawn` to take it down), with your wallet as `author`. Check progress with the `claim_status` MCP tool (`identity` = listing id or guild name). One wallet holds one identity.

**Discord and X verification** are optional and per operator. A node shows them as `verified` only if it trusts the verifier (`HUB_TRUSTED_VERIFIERS`); otherwise they read `self-attested, unverified`. Attestations live in the hub, so they carry over to every node that trusts the same verifier.

**Fix-or-remove.** If your listing is flagged (dead invite, missing description, wording outside the rules), fix it through a claim, or it is removed with a signed `withdrawn` update. Ask for removal any time and it is done, no questions asked.

Do not add files to `updates/` and do not edit `hub-manifest.json` yourself. A signer does that.

## Sign or approve (allowlisted signers)

Signers use the node's own tooling. Keys stay on the signer's machine and are passed by path only.

```bash
# In a clone of sa-ecosystem-node, built with npm ci && npm run build
npm run hub-add-update -- --store /path/to/this/repo \
  --id <id> --kind listing --body "<short plain note>" \
  --listing "$(jq -c .listing proposals/<short-id>.json)" \
  --keypair /path/outside/any/repo/signer.keypair.json --approve --dry-run
```

Drop `--dry-run` to write `updates/<id>.json`, update `hub-manifest.json` and commit. Wallet owners can approve without exporting a key: print the approval bytes with `npm run sign-hub-update -- --print-approval <file>`, sign them with the wallet's `signMessage`, and attach the result with `--approve <file> --public-key <address> --signature <hex>`. The node README has the full flow.

Then push, and give node operators the new commit id and manifest hash.

## Run your own node

Any machine with Node.js 22 or newer, or any free Docker host, can run a node.

```bash
git clone https://github.com/PhoenixClaw01/sa-ecosystem-node
cd sa-ecosystem-node
npm install && npm start      # http://127.0.0.1:8787/mcp
```

`docs/hosting.md` in the node repository covers Docker Compose, stdio for desktop MCP clients, Vercel, Railway, Fly and Render. To serve this hub, set these three together (or none):

```bash
HUB_GIT_REPO=https://github.com/PhoenixClaw01/sa-ecosystem-hub
HUB_GIT_COMMIT=<40-character commit you checked>
HUB_STORE_MANIFEST_HASH=<sha256sum of hub-manifest.json at that commit>
```

Choose your own trusted signer list (`HUB_SIGNERS_PATH` or `hub.signerSets`) and approval threshold. You decide whom your node trusts. Optionally set `HUB_TRUSTED_VERIFIERS` to show Discord or X attestations from verifiers you trust as `verified`, and run the optional Discord verifier (`npm run hub-attest`, `identify` scope only, your own Discord app); see `docs/claim-and-verify.md`. Plug any MCP client into your node's `/mcp` (or `/api/mcp` on Vercel) to use all 22 tools.

## Run your own hub

A guild, DAC or project can run its own hub with its own signers.

```bash
# In a clone of sa-ecosystem-node
npm run hub-store-init -- --dir ../my-hub
cd ../my-hub && git init -b main && git add . && git commit -m "Empty hub update store"
npm --prefix ../sa-ecosystem-node run hub-keygen -- --label "My Guild" --out "$HOME/.config/sa-ecosystem-node/my-guild.keypair.json"
```

The key file must hold no funds and must stay outside every repository. Publish the repository on any public git host, then point your node at it with the three pins above. Story opt-ins name the hub's URL, so pick the final URL before anyone signs one.

## Licence

Content you contribute to this hub is published under **CC BY 4.0**. Code goes to the node, which will be **Apache-2.0** when it goes public.

## Code changes

Code changes go to [sa-ecosystem-node](https://github.com/PhoenixClaw01/sa-ecosystem-node), not here. Run `npm run lint`, `npm run typecheck` and `npm test` before you open a pull request.
