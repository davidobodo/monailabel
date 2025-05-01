# MONAI Label Installation Troubleshooting Guide

## Issue Description

When trying to run the `monailabel` command to download apps (e.g., `monailabel apps --download --name radiology --output apps`), you might encounter the following issues:

1. The command fails with an incomplete PYTHONPATH message:

```
Using PYTHONPATH=/Library/Frameworks/Python.framework/Versions:
```

2. The command might return exit code 127 or show other errors related to module execution.

## Root Cause

The issue occurs because the `monailabel` executable script is setting up the PYTHONPATH incorrectly. The script is located at `/Library/Frameworks/Python.framework/Versions/3.11/bin/monailabel` and contains logic to set the PYTHONPATH, but it's not properly configured for all environments.

## Solution

Instead of using the `monailabel` command directly, use the Python module directly with the following command:

```bash
python3 -m monailabel.main apps --download --name radiology --output apps
```

This bypasses the problematic PYTHONPATH setup in the executable script and directly runs the MONAI Label main module.

## Steps to Resolve

1. First, ensure MONAI Label is installed:

```bash
pip install monailabel
```

2. If the standard installation doesn't work, try installing from source:

```bash
pip install git+https://github.com/Project-MONAI/MONAILabel.git
```

3. Instead of using the `monailabel` command, use the Python module directly:

```bash
python3 -m monailabel.main apps --download --name radiology --output apps
```

## Verification

To verify the installation and command are working:

1. The command should complete successfully
2. You should see output similar to:

```
radiology is copied at: /path/to/your/apps/radiology
```

3. The radiology app files should be present in the specified output directory

## Additional Notes

- This solution has been tested on macOS with Python 3.11
- The issue appears to be related to how the `monailabel` executable script handles PYTHONPATH configuration
- Using the Python module directly (`python3 -m monailabel.main`) is a reliable workaround that bypasses the problematic script configuration

## Support

If you continue to experience issues:

1. Check your Python version: `python3 --version`
2. Verify MONAI Label installation: `pip show monailabel`
3. Try reinstalling MONAI Label: `pip uninstall monailabel && pip install monailabel`
4. Check the MONAI Label documentation for any updates or alternative installation methods
