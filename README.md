Standalone `q` Docker Container.

This project creates a Docker container that provides a minimal [kdb+/`q`](https://kx.com/why-kx/) which aims to support many use cases and environments.

# Related Links

 * [Kx](https://kx.com)
     * [Why Kx?](https://kx.com/why-kx/)
     * [Get going with kdb+](https://code.kx.com)
 * [Docker](https://docker.com)

# Issues

 * Until this project is moved to `KxSystems`, you need to replace `kxsys/dockerq` with `coremem/dockerq`, including in the `gce.metadata.user-data` file

# Pre-flight

A Docker hosting environment is required, the following situations are described.

## [Community Edition](https://www.docker.com/community-edition)

    docker pull kxsys/dockerq

## Google Cloud Engine

**N.B.** on-demand is not permitted for use on a third party cloud provider's computer as per the [license agreement](https://ondemand.kx.com/)

An example of [starting a Docker container on a GCE instance](https://cloud.google.com/compute/docs/containers/deploying-containers) is as follows:

    gcloud compute --project=MYPROJECT instances create dockerq \
    	--zone=europe-west1-b --machine-type=f1-micro \
    	--metadata-from-file=user-data=gce.metadata.user-data \
    	--metadata=QLIC_K4=$(openssl base64 -e -A -in k4.lic) \
    	--image=family/cos-stable --image-project=cos-cloud

The metadata can also be stored in the project rather than at the instance level.

# Usage

You should be able to run:

    docker run --rm -it kxsys/dockerq

You can drop straight into `bash` with:

    docker run --rm -it kxsys/dockerq bash

Lastly you can pipe your program in (or look to [`Q_INIT`](#headless) below):

    $ echo 3+3 | docker run --rm -i kxsys/dockerq q -q
    6

## Headless

You can use [environment variables](https://docs.docker.com/engine/reference/run/#env-environment-variables) (or [instance or project level metadata for GCE](https://cloud.google.com/compute/docs/storing-retrieving-metadata)) to handle any interactive component of starting the container.

 * **`Q_INIT`:** [base64 encoded](https://en.wikipedia.org/wiki/Base64) [`tar`](https://en.wikipedia.org/wiki/Tar_(computing)) ([`gzip`](https://en.wikipedia.org/wiki/Gzip) supported) or [`zip`](https://en.wikipedia.org/wiki/Zip_(file_format)) file that will be extracted to `HOME` and will [automatically begin executing `q.q` if present](https://www.kdbfaq.com/how-can-i-have-kdb-automatically-load-q-code-at-startup-in-every-session/)
     * as inside the container [`QLIC=$HOME`](https://code.kx.com/q/tutorials/licensing/#keeping-the-license-key-file-elsewhere) you may include your `kc.lic` (or `k4.lic`) file
         * it is recommended to use [`QLIC_KC`](#on-demand-license) or [`QLIC_K4`](#commercial-license) so to decouple the licensing from your codebase
     * your upper limit for your `Q_INIT` is something short of the output from `getconf ARG_MAX` (inclusive of the base64 encoding overhead)
         * as cloud metadata [GCE limits you to 256kB](https://cloud.google.com/compute/docs/storing-retrieving-metadata#custom_metadata_size_limitations)

If your project code lives in the directory `mycode`, this lets you invoke `q` using:

    docker run --rm -it -e Q_INIT=$(tar -C mycode -c . | gzip -9 | openssl base64 -e -A) kxsys/dockerq

Alternatively, if your project code lives in a ZIP file called `mycode.zip`:

    docker run --rm -it -e Q_INIT=$(openssl base64 -e -A -in mycode.zip) kxsys/dockerq

### On-demand License

**N.B.** not permitted for use on a third party cloud provider's computer as per the [license agreement](https://ondemand.kx.com/)

The following is supported:

 * **`QLIC_KC`:** base64 encoded contents of your `kc.lic` file

This lets you invoke `q` using:

    docker run --rm -it -e QLIC_KC=$(openssl base64 -e -A -in "$QHOME/kc.lic") kxsys/dockerq

### Commercial License

The following is supported:

 * **`Q_USER` (default: `kx`):** username inside the container in which to run `q`
 * **`QLIC_K4`:** base64 encoded contents of your `k4.lic` file

To amend the hostname of the container, you should use the `h` parameter as follows:

    docker run --rm -it -h myhostname ...

# Build

The instructions below are for building your own Docker image. A pre-built Docker image is available on Docker Cloud, if you only want to run the `q` image then you should read ignore this section and follow the [pre-flight](#pre-flight) and [usage](#usage) sections above on how to do this.

You will need [Docker installed](https://www.docker.com/community-edition) on your workstation; make sure it is a recent version.

Check out a copy of the project with:

    git clone https://github.com/KxSystems/dockerq.git

To build locally the project you run:

    docker build -t dockerq -f docker/Dockerfile .

Once built, you should have a local `q` image, you can run the following to use it:

    docker run --rm -it dockerq
