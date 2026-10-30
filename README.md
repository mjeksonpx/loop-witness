# loop-witness

An append-only journal of verification anchors for
[`mjeksonpx/loopagent`](https://github.com/mjeksonpx/loopagent). It holds no code: one issue, one comment per
anchor verdict, nothing else. **The log itself lives in [issue #1](../../issues/1).**

## Why this exists

`loopagent` accepts agent work by proof: a local gate (`scripts/check_loop.py`) prints a verdict, a binding
anchor records the set of dimension verdicts behind it, and that anchor is signed with a dedicated SSH key.
The honest limit of that signature is written into the tool itself: the key lives on the same machine as the
gate, so the signature proves *"the holder of this key wrote this"* — not *"this is what really happened"*.
Anyone with access to that machine could re-mint the anchor to describe a run that never took place and sign
it again; signature verification alone would not notice.

This journal adds the one thing the machine cannot supply: **a timestamp it does not control**. A rewrite now
has to contradict a recorded comment that was published through GitHub's API, at a time GitHub chose.

The limit is stated plainly rather than dressed up: the comment is posted from the same account, so it can
be deleted. That does not make it useless — a deleted entry is visible in the issue's timeline to anyone who
has read the log — but it is tamper-*evidence*, not tamper-proofness. A signature over the anchor plus this
external record together make a silent substitution conspicuous; neither does it alone.

## What a published entry contains

One comment per distinct verdict, posted by `python3 scripts/anchor_witness.py publish`:

```
<!-- LOOP_ANCHOR_WITNESS -->
**LOOP_ANCHOR_WITNESS**

- verdict_hash: `fe0dd8a6d8ff7640`
- signed message, verbatim:

    marker=LOOP_ENGINEERING_PASS_0001
    verdict_hash=fe0dd8a6d8ff7640

- payload sha256: `7e05815293e38da1c534fc5c9549684999934d5a209f15c7fc38c85a43b940ad`
- signer: `SHA256:3tNJnIe2dT7Xset/ly4xEiZWKLqrTvnMs9IQclqPefs` (loop-anchor@example.org, namespace `loop-anchor`)
- signature: `artifacts/loop_gate_verdict.txt.sig` (verify against `configs/allowed_signers`)
```

No code, no tokens, no paths into anyone's home directory, no personal data: the exact bytes that were
signed, their SHA-256, the verdict hash those bytes commit to, and the fingerprint of a public key. The bytes
are quoted rather than referenced on purpose — a reader of this journal has no checkout to resolve a path
against, and `publish` re-hashes its own payload before posting, refusing to describe a signature that is no
longer the one that was made.

## How to verify an entry

```
git clone https://github.com/mjeksonpx/loopagent && cd loopagent
sha256sum artifacts/loop_gate_verdict.txt      # must equal the entry's payload sha256
ssh-keygen -Y verify -f configs/allowed_signers -I loop-anchor@example.org -n loop-anchor \
    -s artifacts/loop_gate_verdict.txt.sig < artifacts/loop_gate_verdict.txt
```

The first command ties the file to the entry; the second proves the signature over those bytes is genuine and
made by the key whose public half is committed in `configs/allowed_signers`. To check that *this* journal
carries the verdict a given checkout holds, the operator runs `python3 scripts/anchor_witness.py
check-external`: it fails with a named reason when the anchor and the published entry disagree, or when the
current verdict was never published.

## Housekeeping rules

- One comment per verdict hash. Publishing the same verdict twice is a no-op by design.
- Entries are never edited and never deleted, including ones that record a stop verdict.
- A comment is added only by `anchor_witness.py publish`; a hand-written entry proves nothing.

## Disclosure about this journal's first entry

The entry for verdict `fe0dd8a6d8ff7640` was first posted in a form that named the signed payload by path
(`artifacts/loop_gate_verdict.txt`) instead of quoting it, which made the entry unusable outside the checkout
that produced it. It was replaced within the same session, before anything depended on it, by the
self-contained form above; the replaced body is preserved verbatim in the pull request against `loopagent`
that introduced this journal. No other entry has been touched.
