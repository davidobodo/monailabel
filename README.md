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

### Option 1: Use Python Module Directly (Recommended)

Instead of using the `monailabel` command directly, use the Python module directly with the following command:

```bash
python3 -m monailabel.main apps --download --name radiology --output apps
```

This bypasses the problematic PYTHONPATH setup in the executable script and directly runs the MONAI Label main module.

### Option 2: Fix the monailabel Executable Script

If you prefer to use the `monailabel` command directly, you can fix the executable script. Here's how:

1. Create a new script called `monailabel_fixed.sh` with the following content:

```bash
#!/bin/bash

# Copyright (c) MONAI Consortium
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#     http://www.apache.org/licenses/LICENSE-2.0
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

set -e
DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")/../.." >/dev/null 2>&1 && pwd)"

# Add the Python site-packages directory to PYTHONPATH
SITE_PACKAGES="/Library/Frameworks/Python.framework/Versions/3.11/lib/python3.11/site-packages"
export PYTHONPATH=$SITE_PACKAGES:$DIR:$PYTHONPATH
echo Using PYTHONPATH=$PYTHONPATH
echo ""

PYEXE=${MONAILABEL_PYEXE:-python}
version=$(${PYEXE} --version 2>&1)
if echo "$version" | grep "Python 2"; then
  echo "Trying python3 instead of python ($version)"
  PYEXE=python3
fi

exec ${PYEXE} -m monailabel.main $*
```

2. Make the script executable:

```bash
chmod +x monailabel_fixed.sh
```

3. Replace the original script (requires sudo):

```bash
sudo mv monailabel_fixed.sh /Library/Frameworks/Python.framework/Versions/3.11/bin/monailabel
```

The key changes in the fixed script are:

- Added explicit path to Python site-packages
- Modified PYTHONPATH to include site-packages directory first

## Steps to Resolve

1. First, ensure MONAI Label is installed:

```bash
pip install monailabel
```

2. If the standard installation doesn't work, try installing from source:

```bash
pip install git+https://github.com/Project-MONAI/MONAILabel.git
```

3. Choose one of the two solutions above:
   - Either use the Python module directly
   - Or fix the monailabel executable script

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

## Download Dataset

Kindly use this command to download the dataset

```
python3 -m monailabel.main datasets --download --name Task09_Spleen --output datasets
```

## Launching Server

```
python3 -m monailabel.main start_server --app apps/radiology --studies datasets/Task09_Spleen/imagesTr --conf models deepedit
```
