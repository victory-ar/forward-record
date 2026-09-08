# VAR forward record

Public, append-only record of Victory Analytics and Research's pre-registered picks and graded
results, mirrored from the platform at the moment each item is published on
[victory-ar.com](https://victory-ar.com). This repository exists so that anyone can check two claims
the site makes: that every pick was recorded **before** kickoff, and that nothing was edited or
dropped afterwards.

## Layout

| Path | Written when | Contents |
|---|---|---|
| `nfl/<season>/picks/week-NN/<slug>.json` | the research post publishes (hours before kickoff) | one PRIME-tier pick: model and market lines, edge, kickoff, link to the research post |
| `nfl/<season>/tracking/<date>.json`, `latest.json` | Mondays after grading | the forward-tracking payload behind `/performance`: every pick with its result, running accuracy and 95% interval |
| `ufc/<year>/cards/<date>-<slug>.json` | Mondays after the card is graded | every bout the production model priced before the event started: pre-fight probability, de-vigged market, result |
| `ufc/<year>/tracking/<date>.json`, `latest.json` | Mondays | the UFC live-record payload behind `/performance` |
| `anchors/<commit>.json` | the next commit | Sigstore Rekor receipt for that commit (see below) |
| `KEY.pub.pem` | once | the P-256 public key the receipts are signed with |

Files are never rewritten in place except `latest.json` pointers; history is the record.

## How to verify a timestamp

Git alone does not prove *when* something was committed: author dates are self-set and history
can be rewritten. So every commit hash here is logged in
[Sigstore Rekor](https://docs.sigstore.dev/logging/overview/), a public append-only transparency
log that VAR does not control. The logged artifact is the SHA-256 of the commit hash written as
ASCII, signed with the key in `KEY.pub.pem`. Rekor's `integratedTime` is the independent
timestamp: the commit, and therefore every file it contains, provably existed by then.

```bash
COMMIT=<40-hex commit sha>
H=$(printf '%s' "$COMMIT" | sha256sum | cut -d' ' -f1)
rekor-cli search --sha "sha256:$H"                    # -> entry UUID(s)
rekor-cli get --uuid <uuid>                           # integratedTime, and the public key used
# or without rekor-cli:
curl -s -X POST https://rekor.sigstore.dev/api/v1/index/retrieve \
     -H 'content-type: application/json' -d "{\"hash\":\"sha256:$H\"}"
```

Compare `integratedTime` with the kickoff in the pick file. `anchors/<commit>.json` holds a copy
of each receipt for convenience; Rekor is the proof. The most recent commit's receipt is not yet
in the repo (it is committed with the next publish) but is already retrievable from Rekor.

## What this is not

This is a record of a **pre-registered forward test**, not a claim of edge. The corrected backtest
behind the NFL tiers is break-even or below, and on the UFC live record the market's probabilities
are better calibrated than the model's. Read the
[pre-registration](https://victory-ar.com/methodology/2026-27-predictions), its
[correction](https://victory-ar.com/insights/nfl-2026-27-pre-registration-correction), and the
[Honest Validation Protocol](https://victory-ar.com/methodology/protocol) before citing any number.

Data: CC BY 4.0. Writer: var-platform `shared/public_record.py`.
