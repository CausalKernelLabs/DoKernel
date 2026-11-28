## What this changes

<!-- One or two sentences. Link the issue it closes, if there is one. -->

## How it was checked

<!-- The tests you added or ran, and anything you tried by hand. -->

## Checklist

- [ ] `python -m pytest` passes.
- [ ] New behaviour has a test.
- [ ] No weight files, datasets or training code are added.
- [ ] **Preprocessing:** this does not change what the model sees, and `tests/golden/` is untouched.
  - If it does change it: `VERSION` in `dokernel/preprocessing/contract.py` is raised, the reason is stated above, and a maintainer has agreed in an issue. See CONTRIBUTING.md.
