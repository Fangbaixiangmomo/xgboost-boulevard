# Boulevard Backend Development Workflow

This fork contains the native XGBoost backend changes needed for faithful
Boulevard and BRAT-D training.

## Repository Roles

- `xgboost-boulevard/` owns the native backend implementation.
- `boulevard/` owns the user-facing Python API.
- `BRATs/` and `ebm-inference-main/` are historical/experimental references and
  should not be modified as part of the XGBoost backend work.

## Build Artifacts

Do not commit generated build artifacts, including:

- `build/`
- `dist/`
- `*.whl`
- `*.dylib`
- `*.so`
- `*.dll`
- `__pycache__/`

The source fork is version-controlled. Release wheels should be generated from
the source fork during a build/release step.

## Backend Rule

Faithful Boulevard or BRAT-D training must use a Boulevard-enabled backend.
The user-facing `boulevard` package must not silently fall back to vanilla
XGBoost when faithful training is requested.

## Development Order

1. Confirm this fork can build locally.
2. Add disabled Boulevard parameters with default behavior unchanged.
3. Implement plain Boulevard training prediction.
4. Add smoke tests for unchanged default behavior and Boulevard mode.
5. Connect the `boulevard` Python frontend to the modified backend.
6. Add BRAT-D dropout only after plain Boulevard is working.

## Development Log

### Phase 0: Workflow Document

Goal: create a small, version-controlled record of the backend/frontend split
before changing XGBoost internals.

Commands:

```bash
cd /Users/longbo/Developer/boulevard-stack/xgboost-boulevard
git status --short --branch
git add inference_dev/boulevard-backend-workflow.md
git commit -m "docs: record Boulevard backend development workflow"
git push -u fork boulevard/brat-d
```

Result:

- Branch: `boulevard/brat-d`
- Remote used for development: `fork`
- Upstream remote kept for official XGBoost: `upstream`
- No source-code behavior changed.

### Phase 1: Local Native Build Check

Goal: verify this fork can build a local CPU-only XGBoost native library before
any Boulevard-specific source changes.

Commands:

```bash
cd /Users/longbo/Developer/boulevard-stack/xgboost-boulevard
cmake -S . -B build -DUSE_CUDA=OFF
cmake --build build -j 8
```

What these commands do:

- `cmake -S . -B build` configures the build from this source tree and keeps
  generated build files inside `build/`.
- `-DUSE_CUDA=OFF` keeps the first development build CPU-only.
- `cmake --build build -j 8` compiles the configured build with 8 parallel jobs.

Observed result:

```text
[100%] Linking CXX shared library /Users/longbo/Developer/boulevard-stack/xgboost-boulevard/lib/libxgboost.dylib
[100%] Built target xgboost
```

Generated native backend:

```text
/Users/longbo/Developer/boulevard-stack/xgboost-boulevard/lib/libxgboost.dylib
```

Git status after build:

```bash
git status --short
```

Result: no tracked changes. Build outputs are already ignored.

### Phase 2: Install Forked Backend Into `boulevard` Development Environment

Goal: make the `boulevard/.venv` Python environment import the locally built
XGBoost fork instead of the PyPI `xgboost` wheel.

Initial check:

```bash
cd /Users/longbo/Developer/boulevard-stack/boulevard
source .venv/bin/activate
python -c "import xgboost; print(xgboost.__file__); print(xgboost.__version__)"
```

Observed before replacement:

```text
/Users/longbo/Developer/boulevard-stack/boulevard/.venv/lib/python3.12/site-packages/xgboost/__init__.py
3.2.0
```

Replacement install:

```bash
pip uninstall -y xgboost
pip install -v ../xgboost-boulevard/python-package
```

Important install log lines:

```text
INFO:xgboost.packager.locate_or_build_libxgboost:Found libxgboost.dylib at /Users/longbo/Developer/boulevard-stack/xgboost-boulevard/lib
INFO:xgboost.packager.build_wheel:Copying /Users/longbo/Developer/boulevard-stack/xgboost-boulevard/lib/libxgboost.dylib -> .../xgboost/lib/libxgboost.dylib
Successfully installed xgboost-3.3.0.dev0
```

Verification:

```bash
python -c "import xgboost, pathlib; print(xgboost.__file__); print(xgboost.__version__); print(pathlib.Path(xgboost.__file__).parent / 'lib')"
find .venv/lib/python3.12/site-packages/xgboost -name "libxgboost*"
```

Observed after replacement:

```text
/Users/longbo/Developer/boulevard-stack/boulevard/.venv/lib/python3.12/site-packages/xgboost/__init__.py
3.3.0-dev
/Users/longbo/Developer/boulevard-stack/boulevard/.venv/lib/python3.12/site-packages/xgboost/lib
.venv/lib/python3.12/site-packages/xgboost/lib/libxgboost.dylib
```

Result:

- `boulevard/.venv` now imports the locally built XGBoost fork.
- The installed Python package contains the fork-built `libxgboost.dylib`.
- No source-code behavior changed yet.

### Next Phase: Disabled Boulevard Parameters

Next commit target:

```text
gbtree: add disabled Boulevard training parameters
```

Planned scope:

- Add a Boulevard-specific parameter struct in `src/gbm/gbtree.h`.
- Register and serialize the parameters in `src/gbm/gbtree.cc`.
- Keep all defaults off and preserve existing XGBoost behavior.
- Do not change training prediction or tree weights in this phase.
