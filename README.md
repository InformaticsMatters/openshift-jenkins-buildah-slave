# An OpenShift Jenkins Slave Agent Image for "buildah"
This is a Jenkins slave image built on the OpenShift Jenkins Maven
slave that adds [buildah] to the image. It should be
suitable for OpenShift Origin v3.6/v3.7 deployments. A matching
**ImageStream** template is also included for the `latest` image.

Providing...

-   [buildah] (See `Dockerfile/BUILDAH_VERSION` for version)
-   [podman]  (See `Dockerfile/PODMAN_VERSION` for version)
-   [skopeo]  (See `Dockerfile/SCOPEO_VERSION` for version)

>   For background material you can refer to the OpenShift documentation for
    their [Jenkins] service and general information on [builds] and image
    streams.

## Building the image
I use, and you might need...

-   Docker Engine `18.03.0-ce`
-   Docker Compose `1.20.1`
    
Build and tag the image...

    $ docker-compose build

## Deploying the image
Here, we'll deploy to the Docker hub. It's free and simple. We just need to
push it using `docker-compose`.

    $ docker login -u informaticsmatters
    [...]
    $ docker-compose push

## Using the image as an OpenShift Jenkins slave
To use the image as an OpenShift Jenkins slave you can use the accompanying
`slave.yaml` template file to create a suitable **ImageStream** using the `oc`
command-line. Assuming you're logged into the OpenShift server and the project
you've into which you've installed Jenkins you can run the following
command to add a suitable image stream.

    $ oc process -f slave.yaml | oc create -f -

This should result in a `buildah-slave` **ImageStream** that can be used as a
source for a Jenkins slave. To employ the image as a slave you refer to it is
as the `agent` in your Jenkins _pipeline_ where you'd normally refer to an
agent:

    agent {
      label 'buildah-slave'
    }

>   You may need to re-deploy Jenkins (_bounce its Pod_) because it only looks
    for slave-based image streams once, during initialisation. So, if you add a
    slave stream after jenkins was started it'll need to be restarted. When the
    image stream has been recognised you should find it on the
    `Jenkins -> Manage Jenkins -> Configure System` page as a new image
    (with your chosen Name) in the `Kubernetes` section.

Once done you will need to ensure that the slave-agent runs with root
privileges (described in the following section).

## Running the slave agent as root
`buildah` needs to run as root but the default security settings for OpenShift
is not to allow containers to run as root. This can, of course, be modified.
In order to successfully run the slave-agent with root privileges you need to
do a number of things: -

1.  Enable `Run in privileged mode` in the appropriate Kubernetes Pod Template.
    If you've successfully used the `slave.yml` and bounced Jenkins you should
    find a _buildah-slave_ named **Kubernetes Pod Template** in the
    **Kubernetes** section of `Manage Jenkins -> Configure System`
    page on Jenkins. The `Run in privileged mode` option is in the _Advanced_
    section of _Containers_.

1.  In OpenShift you should add `privileged` to the service account
    used to create the Jenkins application. This is normally the
    project/namespace name, which you can check via the **DeploymentConfig**'s
    `spec:template:spec:serviceAccount` value; e.g.
    `oc adm policy add-scc-to-user -z ${SERVICE_ACCOUNT} privileged`.
    Do this while in the appropriate project/namespace.

## So, what can I do?
With this agent you will be able to build container images in an OpenShift
Jenkins agent using local storage without a Docker daemon. You use `buildah`
to `build` and `podman` to login to an external image registry (like Docker)
before finally using `buildah` to push your image to the new registry.

A typical workflow, using a Docker format registry and an existing
`Dockerfile`, might be something like this: -

    $ buildah bud -t me/myimage:latest .
    $ podman login --username <me> --password <password> <registry>:5000
    $ buildah push --format=v2s2 me/myimage:latest docker:<registry>:5000/<namespace>/myimage:latest
    $ podman logout <registry>:5000

...where

-   **registry** is the address/IP of the external container registry.
-   **namespace** is the name of a pre-exiting namespace (project)

Alternatively, you can define the image format (if you're using a Dockerfile)
at the `bud` stage, negating the need for the `--format` option on the push
command: -

    $ buildah bud --format docker -f <Dockerfile> -t me/myimage:latest .
      
---

[buildah]: https://github.com/projectatomic/buildah
[builds]: https://docs.openshift.com/container-platform/3.6/architecture/core_concepts/builds_and_image_streams.html
[jenkins]: https://docs.openshift.com/container-platform/3.6/using_images/other_images/jenkins.html
[podman]: https://github.com/projectatomic/libpod
[skopeo]: https://github.com/projectatomic/skopeo
