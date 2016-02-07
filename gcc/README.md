## Quick Example for a C/C++ Worker

**Note**: Be sure you've followed the base [getting started instructions on the top level README](https://github.com/iron-io/dockerworker).

### 1. Install the dependencies:

TODO:

```sh
docker run --rm -v "$PWD":/worker -w /worker iron/node:dev npm install
```

### 2. Build it

```sh
docker run --rm -e "PAYLOAD_FILE=hello.payload.json" -e "YOUR_ENV_VAR=ANYTHING" -v "$PWD":/worker -w /worker iron/gcc:dev g++ hello.C -o hello
```

### 2. Test locally

Now test it locally:

```sh
docker run --rm -it -e "PAYLOAD_FILE=hello.payload.json" -e "YOUR_ENV_VAR=ANYTHING" -v "$PWD":/app -w /app  iron/gcc ./hello
```

The PAYLOAD_FILE environment variable is passed in to your worker automatically and tells you
where the payload file is. Our [client libraries](http://dev.iron.io/worker/libraries/) help you load the special environment variables automatically.

The YOUR_ENV_VAR environment variable is your custom environment variable. There can
be any number of custom environment variables and they can be anything.

It works! And now that it works, we know it will work on IronWorker.

### 3. Package your code

Let's package it up inside a Docker image and upload it to a Docker Registry. Copy the [Dockerfile](https://github.com/iron-io/dockerworker/blob/master/node/Dockerfile) from this repository
and modify the ENTRYPOINT line to run your script, but for this example, it's all ready to go:

```sh
docker build -t USERNAME/hello:0.0.1 .
```

That's just a standard `docker build` command. The 0.0.1 is the version which you can update
whenever you make changes to your code.

Test your image, just to be sure you created it correctly:

```sh
docker run --rm -it -e "PAYLOAD_FILE=hello.payload.json" -e "YOUR_ENV_VAR=ANYTHING" USERNAME/hello:0.0.1
```

### 4. Push it to Docker Hub

Push it to Docker Hub:

```sh
docker push USERNAME/hello:0.0.1
```

### 4. Register your image with Iron

Ok, we're ready to run this on Iron now, but first we have to let Iron know about the
image you just pushed to Docker Hub. Also, you can optionally register environment variables here that will be passed into your container at runtime.

```sh
iron register -e "YOUR_ENV_VAR=ANYTHING" USERNAME/hello:0.0.1
```

### 5. Queue / Schedule jobs for your image

Now you can start queuing jobs or schedule recurring jobs for your image. Let's quickly
queue up a job to try it out.

```sh
iron worker queue --payload-file hello.payload.json --wait USERNAME/hello
```

Notice we don't use the image tag when queuing, this is so you can change versions
without having to update all your code that's queuing up jobs for the image.

The `--wait` parameter waits for the job to finish, then prints the output.
You will also see a link to [HUD](http://hud.iron.io) where you can see all the rest of the task details along with the log output.

Read the API docs to see how to queue jobs from your code or how to schedule them:
http://dev.iron.io/worker/reference/api/

## Private images

If you want to keep your code private and use a [private Docker repository](https://docs.docker.com/docker-hub/repos/#private-repositories), you just need
to let Iron know how to access your private images:

```sh
iron docker login -e YOUR_DOCKER_HUB_EMAIL -u YOUR_DOCKER_HUB_USERNAME -p YOUR_DOCKER_HUB_PASSWORD
```

Then just do everything the same as above.
