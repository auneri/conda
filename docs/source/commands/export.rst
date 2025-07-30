``conda export``
***************

The ``conda export`` command allows you to export conda environments to various file formats.
This command has been significantly enhanced with a plugin-based architecture that supports multiple export formats for different use cases.

.. argparse::
   :module: conda.cli.conda_argparse
   :func: generate_parser
   :prog: conda
   :path: export
   :nodefault:
   :nodefaultconst:

Overview
========

The export command creates portable representations of conda environments that can be shared
with others or used to recreate environments on different machines. The command supports
multiple output formats through a plugin architecture:

**Structured Formats** (cross-platform friendly):
  - ``environment-yaml`` - YAML format (default) [aliases: ``yaml``, ``yml``]
  - ``environment-json`` - JSON format [aliases: ``json``]

**Text Formats** (platform/architecture specific):
  - ``explicit`` - Explicit URLs with ``@EXPLICIT`` header (CEP 23 compliant)
  - ``requirements`` - Requirements format with MatchSpec strings [aliases: ``reqs``, ``txt``]

Format Selection
================

You can specify the export format in three ways:

1. **Explicit format specification** (recommended):

   .. code-block:: bash

      conda export --format=environment-yaml

2. **File extension detection**:

   .. code-block:: bash

      conda export --file=environment.yaml

3. **Default behavior** (outputs YAML to stdout):

   .. code-block:: bash

      conda export

Supported Output Formats
=========================

Environment YAML Format
------------------------

The default format that creates cross-platform compatible environment files.

**Format name**: ``environment-yaml``
**Aliases**: ``yaml``, ``yml``
**Auto-detected filenames**: ``environment.yaml``, ``environment.yml``

.. code-block:: bash

   # Export to stdout
   conda export --format=environment-yaml

   # Export to file
   conda export --file=environment.yaml

   # Alternative using aliases
   conda export --format=yaml
   conda export --format=yml

Example output:

.. code-block:: yaml

   name: myenv
   channels:
     - conda-forge
     - defaults
   dependencies:
     - python=3.9
     - numpy=1.21.0
     - pandas=1.3.0

Environment JSON Format
-----------------------

JSON representation of the environment for programmatic processing.

**Format name**: ``environment-json``
**Aliases**: ``json``
**Auto-detected filenames**: ``environment.json``

.. code-block:: bash

   # Export to stdout
   conda export --format=environment-json

   # Export to file
   conda export --file=environment.json

   # Alternative using alias
   conda export --format=json

Explicit Format (CEP 23)
------------------------

Creates explicit package URLs for exact environment reproduction.

**Format name**: ``explicit``
**Aliases**: None
**Auto-detected filenames**: ``explicit.txt``

.. code-block:: bash

   # Export to stdout
   conda export --format=explicit

   # Export to file
   conda export --file=explicit.txt

Example output:

.. code-block:: text

   # This file may be used to create an environment using:
   # $ conda create --name <env> --file <this file>
   # platform: osx-64
   @EXPLICIT
   https://repo.anaconda.com/pkgs/main/osx-64/python-3.9.7-h88f2d9e_0.tar.bz2
   https://repo.anaconda.com/pkgs/main/osx-64/numpy-1.21.0-py39h2e5f516_0.tar.bz2

Requirements Format
-------------------

Creates a requirements file with MatchSpec strings.

**Format name**: ``requirements``
**Aliases**: ``reqs``, ``txt``
**Auto-detected filenames**: ``requirements.txt``, ``spec.txt``

.. code-block:: bash

   # Export to stdout
   conda export --format=requirements

   # Export to file
   conda export --file=requirements.txt

   # Alternative using aliases
   conda export --format=reqs
   conda export --format=txt

Example output:

.. code-block:: text

   # This file may be used to create an environment using:
   # $ conda create --name <env> --file <this file>
   python=3.9.7
   numpy=1.21.0
   pandas=1.3.0

Common Options
==============

Export Current Environment
---------------------------

Export the currently active environment:

.. code-block:: bash

   conda export

Export Specific Environment
----------------------------

Export a named environment:

.. code-block:: bash

   conda export --name myenv

Export Environment by Path
---------------------------

Export an environment by its path:

.. code-block:: bash

   conda export --prefix /path/to/env

Cross-Platform Compatibility
=============================

For cross-platform sharing, use the ``--from-history`` flag with structured formats:

.. code-block:: bash

   # Export only explicitly installed packages (cross-platform friendly)
   conda export --from-history --format=environment-yaml

   # This excludes dependency packages that might be platform-specific

When **not** to use ``--from-history``:
  - With ``explicit`` format (always uses all packages)
  - With ``requirements`` format (always uses all packages)
  - When you need exact dependency reproduction

File Format Detection
=====================

The command automatically detects the export format based on filename patterns:

.. list-table:: File Detection Patterns and Aliases
   :widths: 25 25 30 20
   :header-rows: 1

   * - Filename
     - Detected Format
     - Format Aliases
     - Description
   * - ``environment.yaml``
     - ``environment-yaml``
     - ``yaml``, ``yml``
     - YAML environment file
   * - ``environment.yml``
     - ``environment-yaml``
     - ``yaml``, ``yml``
     - YAML environment file (alternative extension)
   * - ``environment.json``
     - ``environment-json``
     - ``json``
     - JSON environment file
   * - ``explicit.txt``
     - ``explicit``
     - None
     - Explicit URL format
   * - ``requirements.txt``
     - ``requirements``
     - ``reqs``, ``txt``
     - Requirements format
   * - ``spec.txt``
     - ``requirements``
     - ``reqs``, ``txt``
     - Requirements format

You can use either the full format name or any of its aliases:

.. code-block:: bash

   # These are all equivalent
   conda export --format=environment-yaml
   conda export --format=yaml
   conda export --format=yml

   # These are all equivalent
   conda export --format=requirements
   conda export --format=reqs
   conda export --format=txt

Examples
========

Basic Usage
-----------

.. code-block:: bash

   # Export current environment to YAML (default)
   conda export > environment.yaml

   # Export specific environment
   conda export --name myenv --format=environment-yaml

   # Export with cross-platform compatibility
   conda export --from-history --file=environment.yaml

Advanced Usage
--------------

.. code-block:: bash

   # Export to explicit format for exact reproduction
   conda export --format=explicit --file=explicit.txt

   # Export to requirements format using aliases
   conda export --format=reqs > requirements.txt
   conda export --format=txt --file=spec.txt

   # Export YAML using short alias
   conda export --format=yml --file=environment.yml

   # Export JSON for programmatic processing
   conda export --format=json --file=environment.json

   # Export with custom channels
   conda export --channel conda-forge --format=environment-yaml

Error Handling
==============

The export command will fail in these cases:

- **Empty environments with text formats**: The ``explicit`` and ``requirements`` formats require installed packages
- **Unrecognized filenames**: Files that don't match supported patterns
- **Invalid format names**: Format names that don't exist

For unrecognized filenames, specify the format explicitly:

.. code-block:: bash

   # This will fail
   conda export --file=my-custom-file.xyz

   # This will work
   conda export --file=my-custom-file.xyz --format=environment-yaml

Plugin Architecture
===================

The export functionality is built on a plugin architecture that allows extending
conda with custom export formats. For information on creating custom exporters,
see :doc:`Environment Exporters <../dev-guide/plugins/environment_exporters>`.

See Also
========

- :doc:`conda env export <env/export>` - Traditional environment export command
- :doc:`Managing environments <../user-guide/tasks/manage-environments>` - Environment management guide
- :doc:`Environment Exporters <../dev-guide/plugins/environment_exporters>` - Plugin development guide
