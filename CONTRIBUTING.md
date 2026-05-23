# Contributing

`scripts/contribute.sh` detects whichever phase is open and contributes to it. The steps below are the same either way.

## Prerequisites

Docker is the recommended path (it pins `snarkjs 0.7.6`). Native install instead: `node >= 20`, `snarkjs 0.7.6`, `bash`, `jq`, `sha256sum`.

You need a GitHub account. A GPG key is optional.

## 1. Fork and enter the build env

Fork this repo on GitHub.

Clone your fork, register the original repo as `upstream`, and build the image:

```bash
git clone https://github.com/<YOUR_USERNAME>/zk-resistor-ceremony.git
cd zk-resistor-ceremony
git remote add upstream https://github.com/TONresistor/zk-resistor-ceremony.git
docker build -t zkr-ceremony .
docker run -it --rm --user "$(id -u):$(id -g)" -e HOME=/tmp -v "$PWD:/work" -w /work --network none zkr-ceremony bash
```

All remaining commands run inside the container, except step 4.

## 2. Verify the current head

```bash
./scripts/verify-previous.sh
```

Hash-checks the head against the transcript, runs the snarkjs cryptographic check, and confirms the chain embeds one contribution per slot. If it fails, stop and open an issue.

## 3. Contribute

```bash
./scripts/contribute.sh <YOUR_NAME>
```

`<YOUR_NAME>` is 1-32 chars, alphanumeric / `-` / `_`. The script re-verifies the head, prompts you for ~30 seconds of random keystrokes (once per circuit in phase 2), writes the new file(s), drafts an attestation into `contributors/`, and appends `transcript/contributions.json`.

Type freely while it prompts. No patterns, no passwords. snarkjs mixes your input with `/dev/urandom`.

## 4. Sign and open the PR

`contribute.sh` prints the exact `git` and `gh pr create` commands for your slot. Exit the container and run them on your host, or open the PR from the GitHub web UI.

You may GPG-clearsign your attestation, or just commit with `git commit -S` (a GitHub-verified signature). At least one is expected; PRs with neither get a manual coordinator review.

CI validates the transcript schema, hash-checks your files, runs the snarkjs verify, and checks the attestation. If it fails, read the output, fix, push again.

## What if another PR merges first?

Your contribution is now on a stale parent. Discard it and redo it on the new head:

```bash
git fetch upstream
git checkout -B <your-branch> upstream/main
./scripts/contribute.sh <YOUR_NAME>
git add phase1/ phase2/ contributors/ transcript/contributions.json
git commit -S -m "<your-branch>"
git push --force-with-lease origin <your-branch>
```

`contribute.sh` picks the next free slot automatically, and your open PR updates itself. PRs merge in arrival order, strict FIFO.

## Destroy entropy

If you used Docker, exiting the container is enough: the entropy lived in container memory and never touched disk. If you went native, clear your shell history (`history -c && history -w`).

## Questions

See [docs/faq.md](./docs/faq.md), or open an issue.
