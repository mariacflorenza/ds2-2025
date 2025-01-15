# Coiled memo for students

## Setup

### Coiled account and 2025 class workspace:
- Create a Coiled account with your academic email at:
  https://cloud.coiled.io/signup

- Send your Coiled account login by email to gmaze@ifremer.fr to get access to shared resources from the "class-2025" Coiled workspace.

### Local Python environment

#### For MacOS and Linux systems

- Download the *ds2-coiled-2025-binder* environment definition we created for the class from here:  
  https://github.com/obidam/ds2-2025/blob/main/practice/environment/coiled/environment-coiled-pinned-binder.yml

- Install and activate this Python 3.11 environment with a package manager like miniconda:  
```bash
miniconda env create -f environment-coiled-pinned-binder.yml
miniconda activate ds2-coiled-2025-binder
```

#### For Windows system

- Download the *ds2-coiled-2025-binder-windows* environment definition we created for the class from here:  
  https://github.com/obidam/ds2-2025/blob/main/practice/environment/coiled/environment-coiled-pinned-binder-windows.yml

- Install and activate this Python 3.11 environment with a package manager like miniconda:  
```bash
miniconda env create -f environment-coiled-pinned-binder.yml
miniconda activate ds2-coiled-2025-binder
```

- Finally, since the google-cloud-sdk library is not available with a package manager, it must be installed manually, and you need to install the specific release 502 from:
  https://storage.googleapis.com/cloud-sdk-release/google-cloud-cli-502.0.0-windows-x86_64.zip

- Note that we won't actually use the Google Cloud SDK from the notebooks, but it is required by Coiled to setup the Dask cluster on the Google Cloud Perform. So, you don't need to configure the SDK with ``gcloud init``.

### Connect to Coiled

- You now need to connect to your Coiled account with the appropriate workspace:
```bash
coiled login --workspace class-2025
```

- Note that you don't have to follow the Step 2 "Connect your cloud" of the Coiled *Get Started* workflow.  
  Only the Step 1 "Authenticate your compute" must be validated.

- If you need packages that are missing from the environment, send an email to gmaze@ifremer.fr to ask for its addition and installation.  
  The environment *ds2-coiled-2025-binder* must be updated and uploaded to all cluster workers on the GCP.


## Usage

### Connect to a cluster

```python
import coiled
from dask.distributed import Client
```

You can then connect to one of the existing clusters:

- ``ds2-highcpu-binder``: 160 vCPUs, 3.75 GB of system memory per vCPU
- ``ds2-highmem-binder``: 20 vCPUs, 6.5 GB of system memory per vCPU

Connect to the cluster, and make it available to Dask for your computation
```python
cluster = coiled.Cluster(name="ds2-highmem-binder", workspace="class-2025")
client = cluster.get_client()
```

**Computation examples**
```python
def inc(x):
    return x + 1
future = client.submit(inc, 10)
future.result() # returns 11
```

```python
import dask.array as da
x = da.random.normal(10, 0.1, size=(20000,20000), chunks= (1000,1000))
y = x.mean(axis=0)[::100]
y.compute();
```


### Jupyter notebook

From your computer, you can use Coiled to create a Jupyter Lab instance in the cloud that will synchronise your local files to the cloud instance. For the synchronisation to work, you're required to install the [``mutagen``](https://mutagen.io/documentation/introduction/installation) software for your OS.

```bash
conda activate ds2-coiled-2025-binder
coiled notebook start --sync --workspace class-2025 --idle-timeout '1 hour' --vm-type n1-highmem-2 --name notebook-user --software ds2-coiled-2025-binder
```

Then on the Jupyter Lab instance, you can connect to the class clusters (see above):
```python
import coiled
from dask.distributed import Client
cluster = coiled.Cluster(name="ds2-highmem-binder")
client = cluster.get_client()
```
