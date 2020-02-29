# Running a GPU-enabled docker container in a GCP VM
Prerequisite:
* gcloud is installed locally and the project is assigned to the one you wanted to run.

## 1. Copy the Makefile to the repo you would like to run (Local)
For testing purpose, you can use this repo directly.
## 2. Update the Makefile  (Local)

Update the following inputs if required:

* VM_NAME=`deep-docker-test`
* ZONE=`us-west1-b`
* REMOTE_IMAGE_NAME=`gcr.io/iconic-mit/tf-2.1.0-gpu-test:latest`
* CONTAINER_NAME=`tf-gpu`

## 3. Upload your Dockerfile to the container registry (Local)

Enable Container Registry API first - [link](https://cloud.google.com/container-registry/docs/quickstart?hl=en_GB)

Before you can push or pull images, 
you must configure Docker to use the gcloud command-line tool to authenticate requests to Container Registry. To do so, run the following command (you are only required to do this once):
* `gcloud auth configure-docker`

CD to the current repo directory and run the following command:
* `make build-cloud-image`

It will take some time to build the docker image and push to the registry.
You can check the building progress in the 
[Cloud Build](https://console.cloud.google.com/cloud-build/builds?project=iconic-mit) Dashboard in GCP.
While building the docker image, you can continue with the next steps.

## 5. Create tag in GCP to open up ports (in GCP)
In GCP, go to VPC network -> Firewall rules:
Adding the two tags to open up the ports:
* rule-allow-tcp-6006
* rule-allow-tcp-8888

For the first one, the main settings are:
* Priority:1000
* Direction: Ingress
* Target tags: allow-tcp-6006
* IP ranges: 0.0.0.0/0
* Protocols and ports: tcp:6006 
* Enforcement: Enabled

## 4. Create a VM on GCP (Local)
Run the following command to create a VM. The VM setup is defined the Makefile:
* `make build-vm-gpu`

Once it is running, you can ssh to the VM. Your username defaults to *docker*:
* `make ssh-vm` 

In the VM, you can check the progress of the startup.sh:
* `cd /var/log`
* `tail -f syslog`

## 5. Pull docker image from the container registry (in VM)
Copy this Makefile to VM (e.g Vim Makefile)

Authorise the docker container inside the VM:
* `gcloud auth configure-docker`

Pull the image to the VM:
* `make pull`

## 6. Create a docker container and check if GPU is working inside the docker (in VM)
We are going to spin up a jupyter notebook container to validate if GPU is working inside the docker.
In the VM, run the following command and copy the notebook URL:
* `make run-exec-notebook`

On your local machine, paste the notebook URL and execute the below python code 
to check if GPU is available:
 ```import tensorflow as tf; tf.test.is_gpu_available()```

## 7. Check how much GPU is used
Type below the check GPU status:
`nvidia-smi`  or `nvidia-smi -l 1`

