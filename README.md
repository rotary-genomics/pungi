# Pungi

A small Python library to help build command-line applications and workflows that utilize the Snakemake workflow management system. It provides utilities for:

- Discovering and organizing sequencing files into Samples and Datasets
- Preparing run directories and YAML configs
- Launching Snakemake with sensible defaults (CPU/memory detection, mamba/conda prefixing)


Overview
--------
Pungi is intended to be embedded into other projects that orchestrate Snakemake pipelines. It does not itself ship a CLI; instead, it exposes functions and classes you can import to:

- Model sequencing files (short/long reads) and sample groupings
- Generate a dataset from a directory or tabular sample sheet
- Write and validate YAML configs
- Invoke Snakemake with common flags and environment setup


Stack and Packaging
-------------------
- Language: Python
- Build/packaging: setuptools (pyproject.toml)
- Dependencies:
  - ruamel.yaml (~=0.17)
  - psutil (~=5.9)
  - snakemake (>=7,<8)
- Package manager: pip (pyproject-based install)
- Entry points / scripts: none currently defined (library-only)


Requirements
------------
- Python: version not explicitly pinned. TODO: Confirm supported Python versions (e.g., 3.8/3.9/3.10/3.11).
- Platforms: not specified. Likely Unix-like systems commonly used for Snakemake. TODO: Confirm Windows support.
- External tools used at runtime when launching workflows:
  - Snakemake (installed as a Python dependency)
  - Conda/mamba for environment management when using Snakemake with `--use-conda` (mamba is the default conda frontend in Pungi's runner). Ensure `mamba` is available on PATH, or adjust flags accordingly.


Installation
------------
Install from a checkout with pip:

- Stable install: `pip install .`
- Editable (development) install: `pip install -e .`

If you need an isolated environment, use your preferred tool (e.g., `conda`, `mamba`, `venv`, or `uv`).


Quick start (library usage)
---------------------------
Below are minimal examples showing how to use the library pieces. Replace paths with your own.

Create a Dataset from a directory of FASTQ files:

```python
from pungi.dataset import generate_dataset_from_fastq_directory

dataset = generate_dataset_from_fastq_directory("/path/to/fastq_dir")
print(list(dataset.identifiers()))
```

Create a Dataset from a sample TSV (see your project’s sample sheet format):

```python
from pungi.dataset import generate_dataset_from_sample_tsv

dataset = generate_dataset_from_sample_tsv("/path/to/samples.tsv", identifier_check=True, integrity_check=True)
for sample in dataset:
    print(sample.identifier(), sample.sample_file_paths())
```

Load, adjust, and write a YAML config:

```python
from pungi.run import load_yaml_config, dump_yaml_config, modify_config_with_available_computational_resources

config = load_yaml_config("/path/to/config.yaml")
config = modify_config_with_available_computational_resources(config)
dump_yaml_config(config, "/path/to/output/config.yaml")
```

Launch a Snakemake workflow:

```python
from pungi.run import run_snakemake_workflow

run_snakemake_workflow(
    config_path="/path/to/output/config.yaml",
    snake_file_path="/path/to/Snakefile",
    output_dir_path="/path/to/run_dir",
    conda_env_dir_path="/path/to/conda_envs",  # where Snakemake will create per-rule envs
    jobs=None,                                   # defaults to os.cpu_count()
    snakemake_custom_args=["--cores", "all"]   # any additional flags
)
```


Scripts and Entry Points
------------------------
- No console scripts or entry points are currently defined in `pyproject.toml`. This is a library intended to be imported by your own CLI or application. TODO: If a CLI is desired, add `console_scripts` entry points under setuptools and document usage.


Environment Variables and Configuration
--------------------------------------
Pungi itself does not define custom environment variables. The following may affect behavior indirectly through Snakemake or the shell environment:

- PATH should include `mamba` (or adjust to use `conda` via custom args) when using `--use-conda`.
- Snakemake-related environment variables (see Snakemake docs) may be respected by Snakemake itself.

TODO: Document any project-specific environment variables if/when they are introduced.


Project Structure
-----------------
Key files and directories:

- `pyproject.toml` — project metadata, dependencies, and build configuration (setuptools)
- `pungi/`
  - `__init__.py`
  - `dataset.py` — Dataset creation and sample TSV handling
  - `sample.py` — Sequencing file, sample, and integrity/identifier utilities
  - `run.py` — YAML config helpers and Snakemake launcher utilities
  - `utils.py` — general helpers (paths, gzip, CLI arg handling)
- `LICENSE` — BSD 3-Clause
- `README.md` — this document
- `meta.yml` — present in repo; contents not documented here. TODO: Clarify its role (e.g., conda recipe?).


Testing
-------
No tests are included in this repository at present. TODO: Add tests (e.g., pytest) covering:

- Dataset/sample parsing and integrity checks
- YAML load/dump and config validation logic
- Snakemake invocation helper behavior (mocked subprocess)


Contributing
------------
Contributions are welcome. TODO: Add contributing guidelines and code style checks if desired.


License
-------
BSD 3-Clause License. See the LICENSE file for details.
