# Coiled memo for students

## Setup

### Coiled account and 2025 class workspace:
- Create a Coiled account with your academic email at:
  https://cloud.coiled.io/signup

- Send an email to gmaze@ifremer.fr to get access to shared resources from the "class-2025" Coiled workspace.

### Local Python environment

- Download the *ds2-coiled-2025-binder* environment definition we created for the class from:  
  https://github.com/obidam/ds2-2025/blob/main/practice/environment/coiled/environment-coiled-pinned-binder.yml

- Install and activate this Python 3.11 environment to easily connect to your Coiled account and GCP clusters:  
```bash
miniconda env create -f environment-coiled-pinned-binder.yml  
miniconda activate ds2-coiled-2025-binder
```

- Connect to your Coiled account:  
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

- ``ds2-highcpu``: 128 vCPUs, 0.9 GB of system memory per vCPU
- ``ds2-highmem``: 64 vCPUs, 6.5 GB of system memory per vCPU

Connect to the cluster, and make it available to Dask for your computation
```python
cluster = coiled.Cluster(name="ds2-highmem", workspace="class-2025")
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
coiled notebook start --sync --workspace class-2025 --idle-timeout '1 hour' --vm-type n1-highmem-2 --name notebook-gmaze --software ds2-coiled-2025-binder
```

Then on the Jupyter Lab instance, you can connect to the class clusters (see above):
```python
import coiled
from dask.distributed import Client
cluster = coiled.Cluster(name="ds2-highmem")
client = cluster.get_client()
```
