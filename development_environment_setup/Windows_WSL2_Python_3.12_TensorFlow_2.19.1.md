# 1. Establish a development environment for using TensorFlow
Date: 2026-03-31  

## TensorFlow's official install page

`TensorFlow` is "An end-to-end platform for machine learning." Before using `TensorFlow`, we need to establish a development environment.

In `TensorFlow`'s install page ( https://www.tensorflow.org/install ), there are three ways to apply TensorFlow 2.
1. Install `TensorFlow` with `pip`
2. Run a `TensorFlow` container
3. Google Colab

Everyone can establish and apply `TenserFlow` in an appropriate way.

# 2. My way: Install `TensorFlow` with `pip` + Windows WSL2 

Here, I provide my approach to establishing an environment for using `TensorFlow`. In this guide, troubleshooting is also included. Please read all before action.  
REF: https://www.tensorflow.org/install/pip

## 2.1. Brief information
My notebook's basic hardware and software specs:  
OS: Windows 11  
GPU: NVIDIA GeForce RTX 3070  
Python: 3.12  
TensorFlow: 2.19.1

## 2.2. Step-by-step installation

"Install `TensorFlow` with `pip` + Windows WSL2" might cause problems for beginners, since this is a cross-platform installation process. A major problem is that the official guides sometimes don't clearly mention the target platform.

In this guide, the target platform will be mentioned first, and then the corresponding actions will be given.

### 2.2.1. Install `Microsoft Visual C++ Redistributable for Visual Studio 2015, 2017 and 2019`
System: In Windows host system  
Windows Native Requires [Microsoft Visual C++ Redistributable for Visual Studio 2015, 2017 and 2019](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist)  
Please install it.

### 2.2.2. Install NVIDIA® GPU drivers
System: In Windows host system  
The following NVIDIA® software are only required for GPU support.  
[NVIDIA® GPU drivers](https://www.nvidia.com/drivers)  
  ->= 528.33 for WSL on Windows  
Please install it (if we want to use GPU)

### 2.2.3. Install WSL2
System: In Windows host system  
Install `WSL2` https://learn.microsoft.com/en-us/windows/wsl/install  
The WSL (Windows Subsystem for Linux) is installed in Windows host system. Hence, we open the "PowerShell" or "CMD" to apply the WSL commands.  
Install WSL:  
```
wsl --install Ubuntu
```
Other functions, please see https://learn.microsoft.com/en-us/windows/wsl/install

### 2.2.4. Setup NVIDIA® GPU support in WSL2
System: In WSL2-Linux (now we go to the Linux system)  
[Setup NVIDIA® GPU support in WSL2](https://docs.nvidia.com/cuda/wsl-user-guide/index.html)  
Read **Option 1: Installation of Linux x86 CUDA Toolkit using WSL-Ubuntu Package - Recommended**, and choose an appropriate installer.

Here, the following selection is chosen to install "CUDA Toolkit 13.2"  
Operating System: Linux  
Architecture: x86_64  
Distribution: WSL-Ubuntu  
Version: 2.0  
Installer Type: deb (local)


### 2.2.5. Install Python3 and establish python virtual environment
System: In WSL2-Linux  
Install `Python3`  
```
sudo apt-get install -y python3-venv python3-pip
```

Establish python virtual environment, activate the environment, and upgrade `pip`  
```
python3 -m venv ml
source ml/bin/activate
pip install --upgrade pip
```

### 2.2.6. Install `TensorFlow` for GPU users
System: In Python3 venv of WSL2-Linux  
Install `TensorFlow` for GPU users  
```
pip install tensorflow[and-cuda]
```

### 2.2.7. Verify the installation
Verify the CPU setup:
```
python3 -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
```
If a tensor is returned, you've installed `TensorFlow` successfully.

Verify the GPU setup:
```
python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
```
If a list of GPU devices is returned, you've installed `TensorFlow` successfully.  
Example message:
```
[PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```

# 3. Troubleshooting during GPU verification
System: In Python3 venv of WSL2-Linux

During GPU verify process, command:  
```
python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
```

Condition: see error message:
```
Cannot dlopen some GPU libraries.  
Skipping registering GPU devices...  
[]
```
This means that `TensorFlow` cannot find the CUDA/cuDNN GPU libraries in WSL2, so only the CPU is detected.
This is usually due to an incomplete WSL2 GPU environment installation or version incompatibility.

Action: check Python version
```
python3 --version
```
My Python version is 3.12

Solution: 
Step 1: Install `TensorFlow` that supports Python 3.12, a stable version of `TensorFlow` is 2.19.1
```
pip uninstall tensorflow -y
pip install tensorflow==2.19.1
```

`TensorFlow` supported table:

| TensorFlow | Supported Python |
| ---------- | ---------------- |
| 2.15       | 3.8 – 3.11       |
| 2.16+      | 3.9 – 3.12       |
| 2.19+      | 3.9 – 3.12       |
| 2.21       | 3.10 – 3.12      |

Step 2: Set path
```
export LD_LIBRARY_PATH=/usr/lib/wsl/lib:$LD_LIBRARY_PATH
```

Step 3: Test
```
python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
```

Condition: see error message:
```
failed call to cuInit: INTERNAL: CUDA error: UNKNOWN ERROR (303)
```
This error is very typical in the new version of the WSL2+ NVIDIA driver, and usually means:  
WSL did not load the Windows GPU runtime correctly (libcuda bridge is not working)  
In other words, the driver is OK, but the WSL GPU runtime is not loaded.

Solution:  
Restart WSL  
In Windows host system
```
wsl --shutdown
wsl
```
Condition: see correct message
```
[PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```
This means that TensorFlow has successfully detected the GPU, and WSL2 + CUDA + `TensorFlow` are all running normally.

# 4. Use `TensorFlow` via `JupyterLab`
 
Use `TensorFlow` via `JupyterLab` in Python3 venv of WSL2-Linux  

1. Create and activate the Python virtual environment  
System: In WSL2-Linux  
```
python3 -m venv ml        # Create the Python virtual environment: ml
source ml/bin/activate    # Activate the Python virtual environment: ml
```

2. Go to the `JupyterLab` working directory  
System: In Python3 venv of WSL2-Linux  
```
cd <JupyterLab working directory>
jupyter lab               # Open Jupyter Lab
```
Note: If `JupyterLab` is not installed, please install it before using it.

Now we can use `TensorFlow` via `JupyterLab` in Python3 venv of WSL2-Linux.

# 5. Install related libraries/applications
System: In Python3 venv of WSL2-Linux  
Environment based installation

Install `JupyterLab`  
REF: https://jupyter.org/
```
pip install jupyterlab
```

Install `scikit-learn`  
REF: https://scikit-learn.org/stable/  
```
pip install scikit-learn
```

Install `pandas`  
REF: https://pandas.pydata.org/
```
pip install pandas
pip install "pandas[excel]"
```

Install `numpy` (this application will be installed while installing `scikit-learn`)  
REF: https://numpy.org/  
```
pip numpy
```

Install `matplotlib`  
REF: https://matplotlib.org/  
```
pip install matplotlib
```

Install `seaborn`
```
pip install seaborn
```
