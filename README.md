Standalone q Docker Container.

This project creates a Docker container that provides a [barebones kdb+/`q`](https://kx.com/why-kx/) which aims to support many use cases and environments.

# Related Links

 * [Kx](https://kx.com)
     * [Why Kx?](https://kx.com/why-kx/)
     * [Get going with kdb+](https://code.kx.com)
 * [Docker](https://docker.com)

# Preflight

A Docker hosting environment is required, the following situations are described.

## [Community Edition](https://www.docker.com/community-edition)

    docker pull kxsys/q

## [Google Cloud Engine](https://cloud.google.com/compute/docs/containers/deploying-containers)

...

# Usage

You should be able to run:

    docker run --rm -it kxsys/q

You can drop straight into `bash` with:

    docker run --rm -it kxsys/q bash

Lastly you can pipe your program in (or look to `Q_INIT` below):

    echo 3+3 | docker run --rm -i kxsys/q q -q

## Headless

You can use [environment variables](https://docs.docker.com/engine/reference/run/#env-environment-variables) (or [metadata for GCE](https://cloud.google.com/compute/docs/storing-retrieving-metadata)) to handle any interactive componment of starting the container.

 * **`Q_INIT`:** [base64 encoded](https://en.wikipedia.org/wiki/Base64) [`tar`](https://en.wikipedia.org/wiki/Tar_(computing)) ([`gzip`](https://en.wikipedia.org/wiki/Gzip) supported) or [`zip`](https://en.wikipedia.org/wiki/Zip_(file_format)) file that will be extracted to `HOME` and will [automatically begin executing `q.q` if present](https://www.kdbfaq.com/how-can-i-have-kdb-automatically-load-q-code-at-startup-in-every-session/)
     * you may wish to include your `kc.lic` (or `k4.lic`) file
     * your upper limit for your `Q_INIT` is something short of the output from `getconf ARG_MAX` (inclusive of the base64 encoding overhead)
     * your upper limit for your `Q_INIT` as cloud metadata is limited by [GCE to the maximum size of 256kiB](https://cloud.google.com/compute/docs/storing-retrieving-metadata#custom_metadata_size_limitations)

If your project code lives in the directory `mycode`, this lets you invoke `q` using:

    docker run -it --rm -e Q_INIT=$(tar -C mycode -c . | gzip -9 | openssl base64 -e -A) q

Alternatively, if your project code lives in a ZIP file called `mycode.zip`:

    docker run -it --rm -e Q_INIT=$(openssl base64 -e -A -in mycode.zip) q

### On-demand License

**N.B.** not permitted for use on a third party cloud provider's computer as per the [license agreement](https://ondemand.kx.com/)

The following is supported:

 * **`QLIC_KC`:** base64 encoded contents of your `kc.lic` file

This lets you invoke `q` using:

    docker run -it --rm -e QLIC_KC=$(openssl base64 -e -A -in "$QHOME/kc.lic") kxsys/q

### Commercial License

The following is supported:

 * **`KDB_USERNAME` (default: `kx`):** username inside the container in which to run `q`
 * **`KDB_HOSTNAME` (default: inherits from host):** hostname inside the container to use
 * **`QLIC_K4`:** base64 encoded contents of your `k4.lic` file

# Build

The instructions below are for building your own Docker image. A prebuilt Docker image is available on Docker Cloud, if you only want to run the `q` image then you should read ignore this section and follow the preflight and usage sections above on how to do this.

You will need [Docker installed](https://www.docker.com/community-edition) on your workstation; make sure it is a recent version.

Check out a copy of the project with:

    git clone https://github.com/KxSystems/q.git

To build locally the project you run:

    docker build -t q -f docker/Dockerfile .

Once built, you should have a local `q` image, you can run the following to use it:

    docker run -it --rm q
