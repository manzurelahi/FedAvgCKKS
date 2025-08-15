# Step-by-Step Guide: CIFAR-10 FedAvg + CKKS (Minimized Setup)

This guide describes the setup and execution of a minimized CIFAR-10 Federated Learning (FL) experiment using NVFLARE, 
organized under the `cifar10-fedavg-ckks` folder. The procedure involves provisioning a secure workspace, 
starting the server and clients, and submitting FL jobs with and without Homomorphic Encryption (HE).

## 1. Assumptions and Setup Scope
We have installed NVFLARE in a Python virtual environment and prepared a minimized project organized under `cifar10-fedavg-ckks`. 
The workflow uses NVFLARE provisioning, an admin client to submit jobs, server-side TensorBoard metric streaming, 
and a homomorphic encryption (HE) job variant.

## 2. Activate the Virtual Environment
```bash
source nvflare-env/bin/activate
```
We have installed and will run everything from this environment.

## 3. Move to the Project Root and Set PYTHONPATH
```bash
cd cifar10-fedavg-ckks
export PYTHONPATH=${PWD}/..
```
This ensures that local modules are discoverable during execution.

## 4. Upgrade Pip and Install Requirements
```bash
pip install --upgrade pip
pip install -r ./requirements.txt
```
This installs the dependencies needed for training and job submission.

## 5. Prepare the CIFAR-10 Data
```bash
./prepare_data.sh
```
This script creates local train/validation splits expected by the training jobs.

## 6. Provision a Secure Workspace
```bash
cd ./workspaces
nvflare provision -p ./secure_project.yml
cp -r ./workspace/secure_project/prod_00 ./secure_workspace
cd ..
```
Provisioning produces a runnable server + clients + admin setup; the generated workspace is copied to `secure_workspace` for convenience.

## 7. Confirm Faster Experiment Setting
Open the following file in the secure workspace:
```
secure_workspace/app_server/config_fed_server.json
```
Verify that:
```json
"min_clients": 2
```
This modification enables quicker job completion during testing.

## 8. Start the Federated System (Server + 2 Clients)
```bash
./start_fl_secure.sh 2
```
This launches the server and two clients inside the secure workspace. Keep this terminal running.

## 9. Submit Experiments from the Admin CLI
Open a new terminal in the project root (still in the virtual environment) to submit jobs.

### 9.1 FedAvg with TensorBoard Streaming
```bash
./submit_job.sh cifar10_fedavg_stream_tb 1.0
```
This job streams metrics to the server during training for central monitoring.

### 9.2 FedAvg with Homomorphic Encryption (CKKS)
```bash
./submit_job.sh cifar10_fedavg_he 1.0
```
This variant applies CKKS-based HE to protect model updates during federated aggregation.

## 10. Shutdown
Once experiments finish, stop the jobs from the admin console (if applicable) and terminate the server/clients by pressing `Ctrl+C` in the `start_fl_secure.sh` terminal.
