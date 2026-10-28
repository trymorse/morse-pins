# Morse pins

These are the enclave measurements Morse's apps accept. Before the Morse app sends a sealed request, it checks the enclave's attestation against these files and refuses an enclave they do not list. This repository is a copy that Morse publishes so that anyone can see what the apps accept and when that changed.

Each commit is made by a bot when Morse's private pins change. Its message says what was pinned or revoked.

## Files

| File | What it pins |
|---|---|
| `enclave_pins.json` | Morse's own AWS Nitro enclave. One entry per production image, with its PCR0, PCR1, and PCR2. An entry with `revoked: true` is refused. |
| `phala_pins.json` | The Phala gateway: its identity, the releases and KMS builds it may run, and the dstack OS images it may run on. |
| `near_pins.json` | The NEAR AI Cloud gateway and model workloads: app ids, KMS roots, TD attributes, the Intel TCB statuses accepted, and the composes, OS images, and instances seen. |
| `phala_model_pins.json` | The container images Phala's model servers ran, one entry per image digest. It records what ran. It does not approve anything. |
| `composes/<compose_hash>` | The measured `app_compose` of each pinned Phala gateway release, named by its sha256. |

## Check Morse's enclave

Each entry in `enclave_pins.json` names its source by `tag`, a tag of [morse-enclave](https://github.com/trymorse/morse-enclave). Check out that tag, rebuild the image as its README describes, and compare your PCR0, PCR1, and PCR2 with the entry's.

Images that Morse ran before it began publishing its enclave source are listed too. Their tags do not exist in morse-enclave, so they cannot be rebuilt from it.

This repository does not take pull requests. To report a security problem, follow [morse-enclave's SECURITY.md](https://github.com/trymorse/morse-enclave/blob/main/SECURITY.md).

## License

The `*_pins.json` files are released under the MIT License. See `LICENSE`. The files in `composes` are Phala's release manifests, copied as measured. Morse did not write them, so `LICENSE` does not cover them.
