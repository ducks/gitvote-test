# GitVote Signed Election Demo

This repository is a disposable demonstration of
[GitVote](https://github.com/ducks/gitvote). It runs a public, signed color vote
through normal Git branches and pull requests.

The old simulated-signature ballots and generated block files remain available
in Git history. The current election uses the `20260717-draft-1` protocol:

- The administrator signs `election.json`.
- Eligible voters sign ballots with independent Ed25519 keys.
- Pull-request CI reads explicit base and head commits.
- A valid pull request adds exactly one canonical ballot and changes nothing
  else.
- Anyone can clone the repository, validate the signatures, and reproduce the
  tally.

This is a test election with no real-world stakes. Ballots are public; GitVote
does not provide secrecy or anonymity.

## Build GitVote

Until the crate release is available, build the pinned source locally:

```bash
git clone https://github.com/ducks/gitvote.git
cd gitvote
git checkout b2879a845254b462e7e43fff1fdc63acdde10ac6
cargo build --release
export GITVOTE_BIN="$PWD/target/release/gitvote"
```

## Validate the Election

From this repository's `main` branch:

```bash
"$GITVOTE_BIN" validate
"$GITVOTE_BIN" tally
```

An election with no accepted ballots is valid and tallies every choice at zero.

## Cast a Demo Ballot

Two demo voter keys are stored outside this repository. The repository owner can
select one of them without exposing the private key in Git:

```bash
git switch -c vote/alice
"$GITVOTE_BIN" cast \
  --choice purple \
  --key ../gitvote-test-keys/alice.key
git push -u origin vote/alice
```

Open a pull request targeting `main`. CI runs the equivalent of:

```bash
"$GITVOTE_BIN" validate-pr \
  --base <current-main-commit> \
  --head <proposed-vote-commit>
```

After Alice's ballot is merged, repeat from the updated `main` branch with
`../gitvote-test-keys/bob.key`. A voter key can have only one accepted ballot.

## Audit After Merge

```bash
git clone https://github.com/ducks/gitvote-test.git
cd gitvote-test
"$GITVOTE_BIN" validate
"$GITVOTE_BIN" tally
```

The audit needs only the public repository. It does not require administrator or
voter private keys.
