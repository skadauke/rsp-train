# RStudio Server Pro Training Environment for R/Medicine 2020 Intro to R Workshop

Docker image for an Intro to R training environment using RStudio Server Pro. This image is based on [rstudio/rstudio-server-pro](https://hub.docker.com/r/rstudio/rstudio-server-pro). It automatically generates a configurable number (up to 999) training accounts, which can be useful for Intro to R courses such as [Intro to R for Clinicians](https://github.com/skadauke/intro-to-r-for-clinicians-rmed2020) at the [R/Medicine 2020 Virtual Conference](https://www.r-medicine.com).

## Features

The `Dockerfile` extends [rstudio/rstudio-server-pro](https://hub.docker.com/r/rstudio/rstudio-server-pro) by adding:

- The `libxml2-dev`, `vim`, and `git` packages
- The `tidyverse`, `rmarkdown`, `shiny`, `flexdashboard`, and `devtools` R packages

In addition, the startup script accomplishes the following:

- Clones a [GitHub repository with course materials](https://github.com/skadauke/intro-to-r-for-clinicians-rmed2020) and extracts the `exercises` and `solutions` directories.
- Creates a configurable number of users with a configurable prefix, e.g. `train001`, `train002`, ... or `rmed001`, `rmed002`, ... 
    - Passwords are random 6-digit numbers. The seed for random password generation is passed at the time `docker run` is invoked.
    - Course materials are automatically placed inside the home directory of each user.
- Automatically activates and deactivates the supplied RStudio Server Pro license when the docker container is started or stopped.

## Configuration

Note that running the RStudio Server Pro Docker image requires the container to run using the `--privileged` flag and a valid RStudio Server Pro license. The image also expects an `/etc/rstudio` folder which customarily is mounted from a local directory named `server-pro/conf`. 

## Example use

### Build the docker image locally

```bash
docker build -t rsp-train .
```

### Start a single server locally on a Docker container

```bash
# Replace with valid license
export RSP_LICENSE=XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX


docker run --privileged -it \
    -p 8787:8787 -p 5559:5559 \
    -e N_USERS=100 \
    -e PW_SEED=12345 \
    -e RSP_LICENSE=$RSP_LICENSE \
    -v "$PWD/server-pro/conf/":/etc/rstudio \
    rsp-train
```

Open [`http://localhost:8787`](http://localhost:8787) to access RStudio Server Pro. 

### List usernames and passwords

The `create_users_table.R` script is used by the startup script to generate users and can be invoked to create a list of usernames and passwords. 

```
Usage: ./create_users_table.R <user_prefix> <n_users> <pw_seed> <filename>
```

For example, to create the list of 100 users with the same seed as in the `docker run` command above, type:

```bash
./create_users_table.R train 100 12345 users.txt
```

This generates the file `users.txt` which is a tab delimited file listing usernames and passwords.
