# Contributing to DoKernel

Thank you for helping. This page says what belongs in this repository, how to set up and test a change, and the one rule that is stricter here than in most projects.

## What this repository is

It holds the **inference code** for DoKernel: loading released weights, preparing a table, and answering questions with the model.

Welcome here:

- Bug fixes, with a test that fails before the fix.
- Clearer errors, documentation and examples.
- Speed and memory improvements that do not change answers.
- New ways of asking the existing model, built on `intervene()`.

Not part of this repository:

- **Training code and training data.** They are developed elsewhere and are not published. Pull requests that add them will be closed.
- **Weights.** They live on Hugging Face under their own licence; see [WEIGHTS_LICENSE.md](WEIGHTS_LICENSE.md). Never add weight files to a pull request.

If you are unsure whether an idea fits, open an issue before writing code.

## Set up

You need Python 3.11 or newer.

```bash
git clone https://github.com/CausalKernelLabs/DoKernel.git
cd DoKernel
python -m venv .venv
source .venv/bin/activate
pip install -e ".[test]"
```

## Run the tests

```bash
python -m pytest
```

The tests use small untrained networks. They need no weights and no network connection, and they finish in a few seconds. All of them must pass before a pull request is merged; the same tests run automatically on every pull request.

One test file, `tests/test_weights.py`, checks real weights and is skipped unless you point it at a release folder:

```bash
DOKERNEL_WEIGHTS=/path/to/release python -m pytest tests/test_weights.py
```

## The rule: preprocessing is a contract

The released weights were trained on tables prepared in exactly one way. If the code prepares a table differently, the weights still run and still return numbers, but the numbers are wrong and nothing warns the user.

So everything that decides what the model sees is written down in [`dokernel/preprocessing/contract.py`](dokernel/preprocessing/contract.py) and guarded by `tests/test_preprocessing_golden.py`. That test feeds fixed tables through preprocessing and compares the result with stored values in `tests/golden/expected.json`.

**If that test fails on your branch:**

- **You did not mean to change preprocessing.** Then your change has a side effect. Undo it under `dokernel/preprocessing/`.
- **You did mean to.** Then the change needs new weights, and it is not a normal pull request. Open an issue first. A change that is agreed needs all of the following in one pull request:
  1. a higher `VERSION` in `contract.py`,
  2. regenerated stored results: `python tests/golden/regenerate.py --i-am-changing-the-contract`,
  3. a statement in the pull request that the contract changed and why,
  4. a maintainer's approval.

**Never regenerate `expected.json` just to make the test pass.** Never edit `tests/golden/inputs.json`.

Weights record the contract version they were trained under, and loading refuses weights from another version, so a contract change cannot silently reach users.

Changes that are safe: adding a new check that refuses bad input with a clear error, as long as valid tables come out exactly as before.

## The query schema is a contract too

`CausalQuery` in `dokernel/ask/schema.py` is what AI models are asked to fill in, so every integration depends on its field names and meanings. Adding an optional field to `Answer` is safe. Renaming or removing a field of `CausalQuery`, or changing what one means, breaks existing integrations: open an issue first.

The package itself must stay free of any AI provider's library. Examples for a specific provider belong in `docs/`.

## Style

- Match the code around your change: plain names, short functions, docstrings that say what a thing is for.
- Keep the package self-contained. It may import the standard library and the dependencies declared in `pyproject.toml`, nothing else; a test enforces this.
- Tests go in `tests/`, as `unittest.TestCase` classes, and must not need weights or a network connection.

## Pull requests

1. Branch from `main`.
2. Make the change, with tests.
3. Run `python -m pytest`.
4. Open the pull request and fill in the template.

By contributing, you agree that your contribution is licensed under the Apache License 2.0, the licence of this repository.

## Conduct and security

Everyone taking part is expected to follow the [Code of Conduct](CODE_OF_CONDUCT.md). To report a security problem, see [SECURITY.md](SECURITY.md); please do not open a public issue for it.
