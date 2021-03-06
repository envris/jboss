# JBOSS Docker image

This is an example Dockerfile with [JBOSS application server](http://jbossas.jboss.org/).

Adapted from [jboss/wildfly](https://github.com/jboss-dockerfiles/wildfly/).

## Usage

To boot in standalone mode

    docker run -it envris/jboss
    
To boot in standalone mode with admin console available remotely

    docker run -p 8080:8080 -p 9990:9990 -it envris/jboss /opt/jboss/jboss/bin/standalone.sh -bmanagement 0.0.0.0

To boot in domain mode

    docker run -it envris/jboss /opt/jboss/jboss/bin/domain.sh -b 0.0.0.0 -bmanagement 0.0.0.0

## Application deployment

With the JBOSS server you can [deploy your application in multiple ways](https://docs.jboss.org/author/display/WFLY8/Application+deployment):

1. You can use CLI
2. You can use the web console
3. You can use the management API directly
4. You can use the deployment scanner

The most popular way of deploying an application is using the deployment scanner. In JBOSS this method is enabled by default and the only thing you need to do is to place your application inside of the `deployments/` directory. It can be `/opt/jboss/jboss/standalone/deployments/` or `/opt/jboss/jboss/domain/deployments/` depending on [which mode](https://docs.jboss.org/author/display/WFLY8/Operating+modes) you choose (standalone is default in the `envris/jboss` image -- see above).

The simplest and cleanest way to deploy an application to JBOSS running in a container started from the `envris/jboss` image is to use the deployment scanner method mentioned above.

To do this you just need to extend the `envris/jboss` image by creating a new one. Place your application inside the `deployments/` directory with the `ADD` command (but make sure to include the trailing slash on the deployment folder path, [more info](https://docs.docker.com/reference/builder/#add)). You can also do the changes to the configuration (if any) as additional steps (`RUN` command).  

[A simple example](https://github.com/goldmann/wildfly-docker-deployment-example) was prepared to show how to do it, but the steps are following:

1. Create `Dockerfile` with following content:

        FROM envris/jboss
        ADD your-awesome-app.war /opt/jboss/jboss/standalone/deployments/
2. Place your `your-awesome-app.war` file in the same directory as your `Dockerfile`.
3. Run the build with `docker build --tag=jboss-app .`
4. Run the container with `docker run -it jboss-app`. Application will be deployed on the container boot.

This way of deployment is great because of a few things:

1. It utilizes Docker as the build tool providing stable builds
2. Rebuilding image this way is very fast (once again: Docker)
3. You only need to do changes to the base JBOSS image that are required to run your application

## Logging

Logging can be done in many ways. [This blog post](https://goldmann.pl/blog/2014/07/18/logging-with-the-wildfly-docker-image/) describes a lot of them.

## Customizing configuration

Sometimes you need to customize the application server configuration. There are many ways to do it and [this blog post](https://goldmann.pl/blog/2014/07/23/customizing-the-configuration-of-the-wildfly-docker-image/) tries to summarize it.

## Extending the image

To be able to create a management user to access the administration console create a Dockerfile with the following content

    FROM envris/jboss
    RUN /opt/jboss/jboss/bin/add-user.sh admin Admin#70365 --silent
    CMD ["/opt/jboss/jboss/bin/standalone.sh", "-b", "0.0.0.0", "-bmanagement", "0.0.0.0"]

Then you can build the image:

    docker build --tag=envris/jboss-admin .

Run it:

    docker run -it envris/jboss-admin

Administration console will be available on the port `9990` of the container.

## Building on your own

You don't need to do this on your own, because we prepared a trusted build for this repository, but if you really want:

    docker build --rm=true --tag=envris/jboss .

## Image internals [updated Oct 14, 2014]

This image extends the [`jboss/base-jdk:7`](https://github.com/jboss-dockerfiles/base-jdk/tree/jdk7) image which adds the OpenJDK distribution on top of the [`jboss/base`](https://github.com/jboss-dockerfiles/base) image. Please refer to the README.md for selected images for more info.

The server is run as the `jboss` user which has the uid/gid set to `1000`.

JBOSS is installed in the `/opt/jboss/jboss` directory.

## Source

The source is [available on GitHub](https://github.com/envris/jboss).
