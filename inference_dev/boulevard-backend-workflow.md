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