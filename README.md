# Morse pins

These are the enclave measurements Morse's apps accept. Before the Morse app sends a sealed request, it checks the enclave's attestation against these files and refuses an enclave they do not list. This repository is a copy that Morse publishes so that anyone can see what the apps accept and when that changed.

A bot makes each commit when Morse's private pins change. Its message says what was pinned or revoked.

## Files

| File | What it pins |
|---|---|
| `enclave_pins.json` | Morse's own AWS Nitro enclave. One entry per image, with its `environment`, PCR0, PCR1, and PCR2. `prod` images serve Morse's apps, and `dev` images serve Morse's development environment. Morse's web app refuses an entry with `revoked: true`. |
| `phala_pins.json` | The Phala gateway: its identity, the releases and KMS builds it may run, and the dstack OS images it may run on. |
| `near_pins.json` | The NEAR AI Cloud gateway and model workloads: app ids, KMS roots, TD attributes, the Intel TCB statuses accepted, and the composes, OS images, and instances seen. |
| `tinfoil_pins.json` | Tinfoil's confidential model router: its hosts, each router release it may run by its AMD SEV-SNP measurement, the guest policy and VMPL, the lowest TCB accepted, and the chips seen. |
| `phala_model_pins.json` | The container images Phala's model servers ran, one entry per image digest. It records what ran. It does not approve anything. |
| `composes/<compose_hash>` | The measured `app_compose` of each pinned Phala gateway release, named by its sha256. |

## Signed releases

Each release of this repository, tagged `signed-<sequence>-<time>`, carries statements that Morse signed with [Sigstore](https://www.sigstore.dev) keyless signing, each with its bundle, and the vendor pins files the statements hash. Morse's apps fetch the newest release to take new pins without an app update, and accept it only when it passes the checks below.

| Asset | What it holds |
|---|---|
| `vendor.statement.json` | A `morse-vendor-pins-v1` statement: the sha256 of the RFC 8785 (JCS) bytes of `phala_pins.json`, `near_pins.json`, `tinfoil_pins.json` and `phala_model_pins.json`, under `files`, keyed by their path in Morse's private pins repository, `v4/<file>` |
| `dev.statement.json`, `prod.statement.json` | A `morse-enclave-pins-v1` statement per environment: the enclave images of `enclave_pins.json` for that environment |
| `<name>.sigstore.json` | The Sigstore bundle that signs `<name>.statement.json` |
| `phala_pins.json`, `near_pins.json`, `tinfoil_pins.json`, `phala_model_pins.json` | The files `vendor.statement.json` hashes, byte for byte as the release's `main` commit holds them |

Every statement names `pins_commit`, the commit of the private pins repository it was written from, `sequence`, the number of commits on that repository's `main` up to it, which only grows, and `time`, when it was written. To check the vendor statement:

```bash
gh release download --repo trymorse/morse-pins --pattern 'vendor.*' --pattern '*_pins.json'
cosign verify-blob --bundle vendor.sigstore.json \
  --certificate-identity https://github.com/Watermark-Labs/pins/.github/workflows/sign.yml@refs/heads/main \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  --certificate-github-workflow-sha "$(jq -r .pins_commit vendor.statement.json)" \
  vendor.statement.json
jq -r '.files | to_entries[] | "\(.value) \(.key | sub("^v4/"; ""))"' vendor.statement.json |
  while read -r sum file; do
    [ "$(jq -jcS . "$file" | sha256sum | cut -c1-64)" = "$sum" ] && echo "$file matches" || echo "$file DIFFERS"
  done
```

The identity is the only one that signs, and the `--certificate-github-workflow-sha` line checks that the statement names the commit it was signed at. The apps also refuse a statement whose `sequence` is below the one they last accepted, and one whose Rekor entry is older than they allow.

## Check Morse's enclave

Each `prod` entry in `enclave_pins.json` names its source by `tag`, a tag of [morse-enclave](https://github.com/trymorse/morse-enclave). Check out that tag, rebuild the image as its README describes, and compare your PCR0, PCR1, and PCR2 with the entry's. A `dev` entry's `tag` names the unreleased commit it was built from. Morse does not publish that source, so nobody can rebuild a `dev` image.

The file also lists images that Morse ran before it began publishing its enclave source. Their tags do not exist in morse-enclave, so nobody can rebuild them from it.

This repository does not take pull requests. To report a security problem, follow [morse-enclave's SECURITY.md](https://github.com/trymorse/morse-enclave/blob/main/SECURITY.md).

## License

Morse releases the `*_pins.json` files under the MIT License. See `LICENSE`. The files in `composes` are Phala's release manifests, copied exactly as measured. Morse did not write them, so `LICENSE` does not cover them.
