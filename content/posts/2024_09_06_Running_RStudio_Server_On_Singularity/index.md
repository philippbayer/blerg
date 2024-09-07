---
title: "Getting RStudio Server to run on Singularity & SLURM as of 2024"
date: 2024-09-06
description: i ramble about rstudio
---

Everybody loves RStudio. Most people like SLURM. Nobody likes the two together. 

I had a quick call today with somebody who wanted to run RStudio Server on Setonix: SLURM-based HPC is obviously not perfect for running a 'constant' server as every job you request has a wall-time, so every few hours your server goes under. But you have the resource, right? Might as well.

Here's how to run RStudio Server via Singularity as of September 2024 - things change constantly, six months ago this was different.

First, we need to make some folders for RStudio Server to write into. I'll just put them into my /scratch directory for now.

```
cd $MYSCRATCH

mkdir home
mkdir home/var
mkdir home/var/lib
mkdir home/var/run
mkdir home/tmp
mkdir home/$USER
```

Now I have a var folder, a tmp folder, and a folder that will be my home directory.


I have this in my .bashrc to quickly request an interactive job, you'd best get something similar:

```
alias getnode='salloc -p debug --nodes=1 --ntasks=32 \
    --cpus-per-task=1 --mem=58G --account=pawsey0812'
```

I request a node, load the Singularity module, and get any RStudio Docker image (tidyverse or rocker or whatever)

```
singularity pull docker://rocker/rstudio
```

This will create a file called `rstudio_latest.sif`. The file contains our entire file system and all RStudio Server dependencies.

Now comes the *fiddly* part: 

```
singularity exec \
   --bind=./home/var/lib:/var/lib/rstudio-server \
   --bind=./home/var/run:/var/run/rstudio-server \
   --bind=./home/tmp:/tmp  \
   --bind=./home/:/home/$USER/ \
   rstudio_latest.sif rserver --www-port 8787 \
   --www-address 0.0.0.0 --server-user $USER
``` 

Let's look at these arguments step by step.

- --bind=./home/var/lib:/var/lib/rstudio-server - This means that we bind our 'local' home/var/lib we created above into the servers '/var/lib/rstudio-server'. Without this line RStudio Server will try to write into this folder, but Singularity file-systems are read-only, so it will crash.
- --bind=./home/var/run:/var/run/rstudio-server - same deal - RStudio will try to write into that folder.
- --bind=./home/tmp:/tmp - again same deal.
- --bind=./home/:/home/$USER/ - I overwrite the home directory within the Singularity container for my username (environment variable $USER) with a home folder outside on scratch, not inside the Singularity container. That lets me install packages in R, which by default go into the user's home directory.
- --www-port 8787 - this will run the RStudio Server on port 8787. Any number > 1000 should be OK.
- --www-address 0.0.0.0 - this will run the RStudio Server on localhost of the remote node.
- --server-user $USER - by default, RStudio tries to run the server under user `rstudio-server` and crash. This option runs the server via your own user.

This should run without a hitch (famous last words!).

One step remains: we need to tunnel from our local machine to the remote node. 

So *on your own machine*, run this, replacing SERVERNAME by whatever server the RStudio Server runs under - it should be something like `nid001234`.

```
ssh -N -f -L 8888:SERVERNAME:8787 YOUR_USER@setonix.pawsey.org.au
```

This will 'build a tunnel' from your local server, port 8888, to the remote server, port 8787 (the port we chose above! Be careful to use the same here.) All we have to do then is to open our browser, and we should have RStudio Server running at http://localhost:8888. Happy RStudio-ing!

Notes:

- home/, var/, tmp/ etc. have to live somewhere. They'll get deleted on most clusters' scratch directories automatically after 21 or 30 days. This is a nice use-case for a [Singularity overlay file-system](https://docs.sylabs.io/guides/3.5/user-guide/persistent_overlays.html)!
- It's not easy to automate this. You'll always need to copy-paste the node name for the local ssh tunnel. 
- It should work pretty much the same for Jupyter(Hub)
