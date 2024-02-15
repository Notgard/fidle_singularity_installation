# Fidle environnement installation guide
A guide for installing the Fidle AI environnement from Docker container using Singularity

**Fidle Environment Installation through Docker Container**

**Prerequisites:**
1. Install Singularity on your local host machine. Follow instructions at [Singularity Installation Guide](https://singularity-tutorial.github.io/01-installation/).

**Steps to Reproduce:**

1. Build the Docker image as a Singularity from a Singularity recipe on a host where you have root privileges (use `sudo` for `singularity build`).

2. Create a `fidle.def` recipe file with the following content:

```
Bootstrap: docker
From: fidlehub/fidle:latest

%post
chmod -R 777 /notebooks
chmod -R 777 /data
chmod -R 777 /run
```

3. Build the container image using:

```
sudo singularity build fidle_img.sif fidle.def
```

This will create the image from the recipe file in the current directory.

4. Copy the built image to your remote host (e.g., Romeo supercomputer) using SCP:

```
scp fidle_img.sif groubahiefissa@romeologin1.univ-reims.fr:~/fidle
```

5. Log in to Romeo and allocate a Slurm node with a GPU:

```
salloc --gres=gpu:1 --reservation=your_reservation
```

6. Generate a custom Jupyter global config for your remote host (Romeo):

```
jupyter notebook --generate-config
```

Modify the generated `jupyter_notebook_config.py` file to allow all IP addresses:

```
c.NotebookApp.allow_origin = '*' # Allow all origins
c.NotebookApp.ip = '0.0.0.0' # Listen on all IPs
```

7. Find the allocated node on Romeo:

```
squeue --me
```

8. SSH to that node and execute the Singularity image from your Fidle folder:

```
singularity run (--nv for GPU) --pwd /notebooks --writable-tmpfs fidle/fidle_img.sif &
```

This specifies Singularity to run the image from the `/notebooks` directory and the container to be run as a writable file system.

9. Now, using simple port forwarding from the remote host to your local host, access the notebook:

```
ssh -N -L port_on_local:romeo17:jupyter_server_port_on_remote groubahiefissa@romeologin1.univ-reims.fr
```

The Fidle notebooks along with the datasets should now be accessible by accessing the following link from your browser:

[http://localhost:port_on_local](http://localhost:port_on_local)
