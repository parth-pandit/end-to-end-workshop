Your developers are now using the best frameworks and practices in developing their application code, but now they need to package that code up for delivery so that it can be run.  

Increasingly, the most popular way to do that is via containers.  But your developers don't want to build an entire container from scratch, so they search around the internet for the best option.  At last count, there were almost **25,000** container images on Docker Hub that mention the word "Java" in them.  So which one is the right on for your developers to use?  And who is responsible for keeping that image updated with the latest security patches?

Tanzu makes this process much simpler for developers and operators by providing services that standardize the container creation process using best practices customized for the type of application technologies your developers are using.  This capability is built on top of the Cloud Native Buildpacks project from the Cloud Native Computing Foundation.  Vmware provides support for the base images for your developers, and provides patches and updates for bugs and CVEs associated with those images.

And when a patch comes out for a critical issue (Heartbleed, Ghost, Shellshock, etc.), your operators can easily rebase all your images in minutes and then roll out those updates to your clusters.

Let's look at how this works with Tanzu!

With Tanzu Build Service, we will create an **image**. This is a mapping of the Git source repo (or the artifact image) for an application artifact to a container image in our Harbor registry. It is important to note that the artifact provided by the developers does not contain Dockerfiles, Kubernetes manifests, or anything else that requires knowledge of the specific container runtime environment that the application will run in. All of that information will be generated, in an automated and consistent way, during the Tanzu CI/CD process.

```terminal:execute
command: kp image create spring-webdb -c demo-cluster-builder --tag harbor.{{ ingress_domain }}/{{ session_namespace }}/spring-webdb --local-path ~/spring-webdb.jar
session: 1
```

A **build** will be triggered when an image is created, or when the application's source code is updated. Use this command to stream the build logs as you describe the buildpack process.

```terminal:execute
command: kp build logs spring-webdb
session: 1
```

Tanzu Build Service recognizes the source code as Java, and employs a Java Buildpack to compile the source and create a custom container image that runs the application. The container image will include VMware's best practices for container design, including up-to-date runtime depdendencies, Java performance optimization, and security hardening for the container runtime. The application developer does not need to be a containerization expert to produce a secure, performant image.

Tanzu Build Service is an important tool for Day 2 Operations as well. Your cluster operators can update the buildpacks, or the base OS image used in your containers, and TBS will automatically trigger a rebuild of all affected containers in your environment. This means no involvement from the individual developers is needed to keep your container runtimes patched and secure.

We actually told TBS to run that build with an outdated OS image, riddled with security vulnerabilities. Let's see what happens when our platform operator publishes a new base image, triggering a rebuild of the affected container without getting the developer involved:

```terminal:execute
command: kp image patch spring-webdb -c default
session: 1
```

See how the image is quickly rebuilt:

```terminal:execute
command: kp build logs spring-webdb
session: 1
```

And check out the build history of the image. The first build was triggered when we defined the image, the second when we provided a new base OS layer:

```terminal:execute
command: kp build list spring-webdb
session: 1
```