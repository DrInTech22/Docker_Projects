# Multi Stage Docker Build

Multi-staging in a Dockerfile is a technique used to create a smaller, efficient and well-optimized docker image.

How does multi-staging works?
Multi-stage build involves using multiple stages to build and compile your application with each stage adapted for a specific purpose, then copying only necessary artifacts from one stage to another.

A typical multi-stage Dockerfile include;
1. Builder stage: This first stage includes a base image, development tools, dependencies and your source code. This stage compiles and build your application.

2. Runtime stage: This final stage involves using a minimalistic runtime image that contains only the necessary artifacts, libraries, and executable files from the previous stages.

The final stage is where the magic happens. It contains only the runtime and necessary build artifacts needed to deploy and run your application.

Multi-staging ensures efficient and fast deployment, portability and less storage cost.
Implementing this technique, I have drastically reduced a golang docker image from 878mb to 1.83mb.
