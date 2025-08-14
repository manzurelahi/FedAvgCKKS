# Federated Learning Runtime Environment Installation Guide

In our experiment, we have installed **NVFLARE**, the **NVIDIA Federated Learning Application Runtime Environment**. NVFLARE is an open-source, domain-agnostic, and extensible SDK designed for Federated Learning (FL). It provides a secure and privacy-preserving platform for distributed multi-party collaboration.

---

## Prerequisites

Before starting the installation, we ensured the following software was available:

- Python 3.9 or newer  
- pip (Python package installer)  
- Git  

**Note:**  
The server and client versions of NVFLARE must match exactly, as cross-version compatibility is not supported.

---

## Operating System Selection

For our setup, we have chosen **Linux Ubuntu Minimal 22.04 LTS (Jammy Jellyfish)**.

---

## Installation Method

We recommend installing NVFLARE in a **Python virtual environment** to maintain isolation from system-level packages.

### 1. Preparing the Virtual Environment
Python’s built-in `venv` module allows us to create and manage virtual environments.  
On Ubuntu 22.04 Minimal, we installed the `venv` package with:

```bash
sudo apt update
sudo apt-get install python3-venv
```

### 2. Creating the Virtual Environment
Once `venv` was installed, we created the NVFLARE environment:

```bash
python3 -m venv nvflare-env
```

This step created a directory named `nvflare-env` in our working directory, containing a dedicated Python interpreter, standard libraries, and required supporting files.

### 3. Activating the Environment
To use this environment, we ran:

```bash
source nvflare-env/bin/activate
```

---

## Updating Package Tools
After activating the environment, we updated `pip` and `setuptools` to their latest versions:

```bash
python3 -m pip install -U pip
python3 -m pip install -U setuptools
```

---

## 4. Installing NVFLARE (Stable Release)
Finally, we installed the stable release of NVFLARE:

```bash
python3 -m pip install nvflare
```

At this stage, NVFLARE and its dependencies were successfully installed in our isolated Python environment, ready for configuration and use.
