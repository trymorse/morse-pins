# Morse pins

These are the enclave measurements Morse's apps accept. Before the Morse app sends a sealed request, it checks the enclave's attestation against these files and refuses an enclave they do not list. This repository is a copy that Morse publishes so that anyone can see what the apps accept and when that changed.

A bot makes each commit when Morse's private pins change. Its message says what was pinned or revoked.

## Files

| File | What it pins |
|---|---|
| `enclave_pins.json` | Morse's own AWS Nitro enclave. One entry per image, with its `environment`, PCR0, PCR1, and PCR2. `prod` images serve Morse's apps, and `dev` images serve Morse's development environment. Morse's web app refuses an entry with `revoked: true`. |
| `phala_pins.json` | The Phala gateway: its identity, the releases and KMS builds it may run, and the dstack OS images it may run on. |
| `near_pins.json` | The NEAR AI Cloud gateway and model workloads: app ids, KMS roots, TD attributes, the Intel TCB statuses accepted, and the composes, OS images, and instances seen. |
| `phala_model_pins.json` | The container images Phala's model servers ran, one entry per image digest. It records what ran. It does not approve anything. |
| `composes/<compose_hash>` | The measured `app_compose` of each pinned Phala gateway release, named by its sha256. |

## Check Morse's enclave

Each `prod` entry in `enclave_pins.json` names its source by `tag`, a tag of [morse-enclave](https://github.com/trymorse/morse-enclave). Check out that tag, rebuild the image as its README describes, and compare your PCR0, PCR1, and PCR2 with the entry's. A `dev` entry's `tag` names the unreleased commit it was built from. Morse does not publish that source, so nobody can rebuild a `dev` image.

The file also lists images that Morse ran before it began publishing its enclave source. Their tags do not exist in morse-enclave, so nobody can rebuild them from it.

This repository does not take pull requests. To report a security problem, follow [morse-enclave's SECURITY.md](https://github.com/trymorse/morse-enclave/blob/main/SECURITY.md).

## License

Morse releases the `*_pins.json` files under the MIT License. See `LICENSE`. The files in `composes` are Phala's release manifests, copied exactly as measured. Morse did not write them, so `LICENSE` does not cover them.
