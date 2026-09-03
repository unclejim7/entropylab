# Setting up a trusted air-gapped machine

EntropyLab is built to be run on a computer that is never connected to a
network while it handles wallet secrets. This is a step-by-step guide for
turning a spare laptop into that machine. It complements
[README.md's "Verifying the download"](README.md#verifying-the-download);
read that section for the exact verification commands.

None of this is required to use EntropyLab casually. It matters when the
seed phrases, private keys, or extended keys you plan to enter secure real
funds.

## 1. Hardware

Getting real confidence that a machine is both offline and trustworthy
involves more than one setting, and doing it well isn't necessarily easy or
for everyone. No single step below is sufficient on its own — think of them
as levels of assurance that add up, not a checkbox to clear:

- **Prefer a machine with known history and provenance** over an unknown or
  second-hand one, when you have the choice. Provenance doesn't guarantee a
  clean machine — a brand-new laptop isn't automatically trustworthy either
  — but it's one real input into your confidence in the system, not
  irrelevant. For a machine that will handle bitcoin private key material,
  don't discard it.
- **Wipe the disk (not just reformat) and install a verified OS from
  verified media** (see [Section 2](#2-get-the-os-on-a-separate-networked-machine)).
  This addresses disk-level compromise, but not everything: malware or
  implants can live in firmware, UEFI/BIOS, or other components a disk wipe
  never touches. There is no simple, universally-available way to rule that
  out on arbitrary hardware.
- **Physically remove the Wi-Fi/Bluetooth card where practical.** This is
  meaningfully different from disabling radios in the OS or BIOS/UEFI —
  software/firmware settings can be misconfigured, reset, or overridden by
  something that already has enough access to matter. Treat disabling radios
  in BIOS/UEFI and the OS as an additional layer of defense, not as proof
  that the machine has no functioning radio hardware.
- **Keep the machine permanently offline once it's set up.** No software
  update, no "just this once" — see [Section 4](#4-transfer-and-use).
- Cover the camera and microphone if present.

### The limits of an air gap: side-channel and covert-channel leakage

Being offline stops ordinary network exfiltration. It does not stop
information from leaving the machine, or being read off it, through
physical channels — and this applies for as long as you use the machine,
not just during setup.

**If the machine is already compromised** (or something near it is),
malware doesn't need a network to exfiltrate data. It can encode secrets
into channels a network monitor would never see: speaker output (audible or
ultrasonic), keystroke timing, fan speed or hard-drive actuator noise, power
draw on the CPU or PSU (measurable as fluctuations on a shared power line),
electromagnetic emissions from cables or the CPU, blinking status LEDs, or
even heat output — for a nearby microphone, camera, power-line tap, or other
receiving device to pick up. Fully wiping the disk does not address any of
these if the compromise lives in firmware or hardware rather than the OS.

**Even on a clean machine with Wi-Fi/Bluetooth off**, passive physical
leakage doesn't require any prior compromise at all: keystroke acoustics,
electromagnetic emissions from the CPU/display/cables, power-consumption
analysis correlated with cryptographic operations, acoustic emissions from
components like coils and power-supply parts, mechanical vibration, and
optical/EM leakage from the display are all real, studied attack classes.

Most of these require specialized equipment and physical proximity a casual
attacker won't have, and this guide doesn't attempt to defend against all of
them. The point is narrower: **"offline" is not the same claim as "cannot
leak."** If your threat model includes a well-resourced or physically
proximate adversary, treat the steps above as a starting point rather than a
complete answer, and research shielding, enclosures, and physical-isolation
practices separately. See also
[SECURITY.md](SECURITY.md#security-considerations) — "None of this protects
against a compromised machine" applies here too.

## 2. Get the OS on a separate, networked machine

Do the download, verification, and USB flashing on a different, ordinary
computer — never on the air-gapped laptop, which should never see a
network in the first place.

- [Tails](https://tails.net/) is a live, amnesic OS: nothing persists
  between boots, which is a reasonable default for key generation since it
  leaves no trace on disk. A persistent install (e.g. Debian) is only
  needed if you have a reason to keep files across sessions.
- Verify the OS image's checksum and signature against the project's
  official keys before flashing it — see Tails' own
  [verification instructions](https://tails.net/install/expert/index.en.html).
  Do not skip this step; it is the only thing standing between you and a
  tampered installer image.
- Flash the verified image to a USB stick (e.g. with `balenaEtcher` or
  `dd`), then boot the target laptop from it.

## 3. Verify EntropyLab before it ever reaches the air-gapped machine

Do this on the same networked machine, before transferring anything to the
laptop. Two independent ways to get a verified copy of `entropylab.html`,
either is sufficient:

**A. Download and verify the release**

```sh
# from a release or entropylab.online, then in the download directory:
sha256sum -c SHA256SUMS.txt
gh attestation verify entropylab.html -R w-s-bitcoin/entropylab
```

The checksum alone only catches accidental corruption. Pairing it with the
[Sigstore attestation](https://github.com/w-s-bitcoin/entropylab/attestations)
authenticates that the exact bytes came from this repository's release
workflow.

**B. Build it yourself and confirm it's reproducible**

```sh
git clone https://github.com/w-s-bitcoin/entropylab.git
cd entropylab
node --version   # >= 20.19
npm ci
npm run ci       # test:ci + build + verify — same checks CI runs
```

`npm run ci` builds `entropylab.html` from `src/` and checks the result
against the project's invariants (no network egress, no entropy
generation, artifact integrity — see [CONTRIBUTING.md](CONTRIBUTING.md)).
`entropylab.html` is generated and git-ignored, not committed to the repo,
so there is no committed copy to diff your build against — the published
`SHA256SUMS.txt` and attestation in option A are what let you confirm an
independently-built copy matches what CI produced.

For either path, the [development container](CONTRIBUTING.md#the-development-container-no-host-prerequisites)
(`docker compose up --build`) gives you a pinned Node/Rust/browser
environment with no host prerequisites, so the build isn't at the mercy of
whatever happens to be installed on the networked machine.

## 4. Transfer and use

- Copy the verified `entropylab.html` to a USB stick and carry it to the
  air-gapped laptop. Nothing else needs to cross that gap.
- Boot the laptop, confirm networking is off, and open the file in a
  browser.
- Before entering any seed phrase, private key, or other secret, follow
  README's [Usage](README.md#usage) guidance: disconnect all networks and
  verify important addresses and descriptors with an independent wallet or
  signing device before receiving funds. EntropyLab is a calculator, not a
  signer — see [SECURITY.md](SECURITY.md) for the full threat model.
- If the OS is Tails (amnesic), nothing you did persists after shutdown; if
  it's a persistent install, treat the laptop itself as sensitive material
  going forward and store it accordingly.
