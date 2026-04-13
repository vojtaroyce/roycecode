# Analytical Mode

The legacy Python analytical-mode flow has been removed from the repository.

The current Rust CLI exposes deterministic analysis and architecture-surface
generation only:

```bash
roycecode analyze /path/to/project
roycecode surface /path/to/project
```

If analytical or policy-tuning workflows return, they should be implemented as
Rust-native features rather than restored as Python-side tooling.
