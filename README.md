# loop-witness

An append-only journal of verification anchors for
[`mjeksonpx/loopagent`](https://github.com/mjeksonpx/loopagent). It holds no code: one issue, one comment per
anchor verdict, nothing else.

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
- signed message: `marker=LOOP_ENGINEERING_PASS_0001\nverdict_hash=fe0dd8a6d8ff7640` (sha256 `7e05815293e38da1…`)
- signer: `<key fingerprint>` (loop-anchor@example.org, namespace `loop-anchor`)
- signature: `artifacts/loop_gate_verdict.txt.sig`
```

No code, no tokens, no paths, no personal data: a hash of the verdict set, the exact bytes that were signed,
the hash of those bytes, and the fingerprint of a public key.

## How to verify an entry

```
git clone https://github.com/mjeksonpx/loopagent && cd loopagent
ssh-keygen -Y verify -f configs/allowed_signers -I loop-anchor@example.org -n loop-anchor \
    -s artifacts/loop_gate_verdict.txt.sig < artifacts/loop_gate_verdict.txt
```

That proves the signature over the verdict is genuine and made by the key committed in
`configs/allowed_signers`. To check that *this* journal carries the current verdict, the operator runs
`python3 scripts/anchor_witness.py check-external` — it fails with a named reason when the anchor and the
published entry disagree, or when the current verdict was never published.

## Housekeeping rules

- One comment per verdict hash. Publishing the same verdict twice is a no-op by design.
- Entries are never edited and never deleted, including ones that record a stop verdict.
- A comment is added only by `anchor_witness.py publish`; a hand-written entry proves nothing.
