# CLAUDE.md

Guidance for Claude when working in this repository.
Package name: `PLACEHOLDER` (placeholder, to be renamed later).

---

## 1. Project overview
Check the TODO to see how to replace this overview.

### 1.1 Commands
```bash
conda env create -f environment.yml   # then: conda activate <env>
pip install -e .[tests,docs]
pre-commit install                    # once per clone
pytest                                # hardware tests skipped by default
pytest --cov=PLACEHOLDER --cov-report=term-missing   # local coverage report
pre-commit run --all-files
```

### 1.2 Layout
`src/PLACEHOLDER/` package · `tests/` pytest suite · `doc/` Sphinx docs.

---

## 2. Working rules (highest priority)

These override any default habits.

### 2.1 Ask, don't assume
- If something is ambiguous, **ask the user**. Never fill gaps with guesses.
- This includes naming, file locations, file formats, behavior on errors and units.
- Offer short options with a recommendation when asking.

### 2.2 Scope discipline
- Work **only** on what the current task or issue asks for.
- Do not refactor, rename, reformat or "improve" unrelated code.
- If you notice a bug, a smell, a missing test or an improvement outside the scope, **do not fix it**. Add it to `TODO.md` (see 2.5) and carry on.
- Scope may be widened only when the user says so explicitly.

### 2.3 Minimal code, minimal abstraction
- Write the most concise code that is still readable.
- **Do not introduce new abstractions** (new base classes, wrappers, registries, helper layers, decorators, mixins, config systems) without **explicit user permission**.
- When you think an abstraction is justified, stop and ask first. Explain what it would remove and what it would cost.
- Prefer plain functions and the existing classes. No speculative generality: do not build for features nobody asked for.

### 2.4 Dependencies
- Always ask before adding a dependency.
- If the user approves, add it to **both** `environment.yml` and `pyproject.toml` in the same change.

### 2.5 TODO.md
- Keep a `TODO.md` in the repo root for everything noticed but out of scope.
- Format: one bullet per item with a short description, the file or area, and the date.
- **Mention the contents of `TODO.md` to the user regularly**, at least at the end of every completed task, and whenever new entries were added.

### 2.6 Hardware safety [USER INPUT: ask whether this project uses hardware; remove 2.6 and the hardware lines in section 4 if not]
- **Never run code that talks to real hardware without explicit authorization** from the user. This includes tests marked `hardware`, cooler or temperature changes, shutter or grating moves, and any device `open()` call on a real driver.
- By default, run only virtual-device tests.
- If a task seems to need hardware, say so and wait.

### 2.7 Git workflow
- Do not commit, push, create branches or open PRs on your own.
- Branching, commit style and PR flow are to be discussed with the user. Ask when it comes up.

### 2.8 Check requirements 
From time to time check if the requirements in `pyproject.toml` and `environment.yml` are still up to date; if not, add a note to `TODO.md`.

### 2.9 Data and secrets
- Never commit raw/large data, generated output or credentials. Never modify raw data in place.
- No hard-coded absolute paths; use `pathlib` (code must run on Windows, Linux and macOS).

### 2.10 Definition of done
Tests pass, `pre-commit run --all-files` is clean, public API has docstrings, README/`doc/` updated if the API changed, `TODO.md` updated.

---

## 3. Code style

- **Language:** English for code, comments, docstrings, commits and docs.
- **Formatter:** `black` (default settings, unless `pyproject.toml` says otherwise). Run it before presenting code.
- **Python version:** as set by `requires-python` in `pyproject.toml`.
- **Errors and logging:** use `logging`, not `print`. No bare `except:` or silently swallowed errors.
- **Reproducibility:** seed random generators; store settings and versions alongside results.
- **Type hints:** [DEFAULT] on all public functions and dataclass fields.
- **Units in names** wherever possible: `exposure_s`, `wavelength_nm`, `temperature_c`, `pixel_width_um`.
  In dataclasses, also state the unit in a comment or in the docstring's attribute list.
- Settings dataclasses are `@dataclass(frozen=True)`. Changes go through `apply_settings()` and never through mutation.

### 3.1 Docstrings: numpydoc format

Every public module, class and function gets a numpydoc-style docstring.
Describe *how it works and why*, not just a restatement of the name.

```python
def wavelength_axis(self, n_pixels: int, pixel_width_um: float) -> np.ndarray:
    """Compute the wavelength of every detector pixel.

    The grating equation gives the wavelength at the detector centre; the
    remaining pixels are offset linearly using the calibrated dispersion of
    the current grating, so the result is only valid for the current settings.

    Parameters
    ----------
    n_pixels : int
        Number of pixels along the dispersion axis.
    pixel_width_um : float
        Physical pixel width in micrometres.

    Returns
    -------
    numpy.ndarray
        Wavelength in nanometres, shape ``(n_pixels,)``.

    Raises
    ------
    DeviceError
        If the spectrograph is not open.
    """
```

### 3.2 Comments: explain *how*, not *what*

Inline comments explain the mechanism and the reason. They must not narrate the line.
Non-obvious lines and every loop need a comment explaining what the loop accomplishes and why.

```python
# Bad: restates the code
i += 1  # increment i

# Good: explains the mechanism
# The camera returns frames oldest-first, so we drain the buffer until it is
# empty; stopping early would leave stale frames for the next acquisition.
while (frame := self._read_next()) is not None:
    frames.append(frame)
```

---

## 4. Testing

- Framework: `pytest`.
- **Keep the test count small.** Not everything needs its own test.
  - Merge related checks into one test (several asserts, parametrization) where the failure message stays clear.
  - Before adding a test, check whether an existing one can be extended.
  - Review the suite regularly and merge overlapping tests.
- **Keep coverage high**, checked locally with `pytest-cov` (see 1.1). Few tests, but they should exercise most of the code. No external coverage services.
- [USER INPUT: hardware only, see 2.6] Shared contract tests run against every driver implementation, via `parametrize`.
- [USER INPUT: hardware only, see 2.6] Hardware tests carry the `hardware` marker and are **skipped by default**. Never run them without authorization (see 2.6).
- [USER INPUT: hardware only, see 2.6] Tests must not depend on Andor DLLs. Use the virtual devices.

---