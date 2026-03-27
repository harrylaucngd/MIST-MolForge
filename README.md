# MIST-MolForge

`MIST-MolForge` is a minimal benchmark repository for:

- MIST spectrum-to-fingerprint prediction
- thresholding the predicted 4096-bit fingerprint into discrete on-bits
- upstream MolForge decoding from fingerprint tokens to molecules
- benchmarking on MassSpecGym and CANOPUS

This repo is intentionally narrow. It keeps only the code needed for the
MIST -> thresholded fingerprint -> MolForge -> benchmark path.

## Layout

- `src/mist/`: retained MIST encoder and benchmark data-loading code
- `src/mist_molforge/`: thin integration layer for benchmark orchestration, chemistry helpers, metrics, and the MolForge adapter
- `MolForge/`: upstream MolForge repository, tracked as a git submodule
- `configs/`: MSG and CANOPUS benchmark configs
- `checkpoints/`: local MIST checkpoints, MolForge checkpoint, and SentencePiece models

There is no separate `scripts/` wrapper anymore. The package entrypoint is the
only benchmark interface.

## Upstream Provenance

The `MolForge/` directory is intended to remain an upstream submodule from
[knu-lcbc/MolForge](https://github.com/knu-lcbc/MolForge.git). The integration
layer in this repo uses upstream MolForge model code and beam search rather
than maintaining a separate local reimplementation.

Upstream reference:
[MolForge upstream](https://github.com/knu-lcbc/MolForge.git)

## Setup

Install the local package:

```bash
pip install -e .
```

Optional MCES metrics dependency:

```bash
pip install -e .[mces]
```

If you are initializing the repo with the upstream MolForge submodule:

```bash
git submodule update --init --recursive
```

## Expected Local Artifacts

Dataset side:

- `data/msg/...`
- `data/canopus/...`

Each benchmark config expects processed files such as:

- `labels.tsv`
- split TSVs
- `spec_files/*.ms`
- precomputed subformula JSON files

Checkpoint side:

- `checkpoints/mist_msg.pt`
- `checkpoints/mist_canopus.pt`
- `checkpoints/decoder_molforge.pth`
- `checkpoints/molforge_sp/combined_morgan4096_vocab_sp.model`
- `checkpoints/molforge_sp/combined_smiles_vocab_sp_morgan4096.model`

If you later add dataset-specific MolForge decoder checkpoints, set them in the
config under `molforge.checkpoint`.

## Canonical Benchmark Command

After `pip install -e .`, the single canonical benchmark command is:

```bash
mist-molforge-benchmark --config configs/spec2mol_benchmark_msg.yaml
```

For CANOPUS:

```bash
mist-molforge-benchmark --config configs/spec2mol_benchmark_canopus.yaml
```

Typical GPU usage examples:

```bash
mist-molforge-benchmark \
  --config configs/spec2mol_benchmark_msg.yaml \
  --device cuda \
  --output-dir results/msg
```

```bash
mist-molforge-benchmark \
  --config configs/spec2mol_benchmark_canopus.yaml \
  --device cuda \
  --output-dir results/canopus
```

Useful overrides:

```bash
mist-molforge-benchmark \
  --config configs/spec2mol_benchmark_msg.yaml \
  --thresholds 0.5 0.172 \
  --max-spectra 128 \
  --batch-size 8 \
  --device cuda
```

## Config Structure

The benchmark configs contain only four sections:

- `data.*`: dataset files
- `mist_encoder.*`: MIST encoder architecture and checkpoint
- `molforge.*`: MolForge submodule root, checkpoint, SentencePiece models, and decode settings
- `evaluation.*`: split and optional sample cap

## Notes

- The package entrypoint is `mist_molforge.benchmark:main`.
- This repo does not bundle MSG or CANOPUS data.
- Checkpoints are local runtime artifacts; whether to commit them, ignore them,
  or move them to Git LFS is a separate repository policy decision.
