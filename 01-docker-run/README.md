Once you are done with the docker installation, let's try running the following command:

```sh
docker run hello-world
```

It would fail with an error like:

```sh
docker: failed to connect to the docker API at unix:///var/run/docker.sock; check if the path is correct and if the daemon is running: dial unix /var/run/docker.sock: connect: no such file or directory
Run "docker run -help" for more information.
```

**What just happened?**

We installed Docker Desktop, which is an app that sets up the Docker environment including the Docker client. But in order to run containers, we have to start the Docker Desktop app first.

After running Docker Desktop, let's try again with `docker run hello-world`. This time it should work!

NOTE: docker/containerization is fundamentally a Linux technology that requires a Linux kernel to run natively. When you start Docker Desktop, it automatically starts a lightweight, optimized Linux VM (using WSL 2 on Windows or the Virtualization framework/HyperKit on Mac) in the background. This VM hosts the actual Docker Engine. There is a lot more to discuss about it that we will cover later.

## Understanding Image Names

When you run `docker run hello-world`, Docker actually looks for the image at `docker.io/library/hello-world:latest`. The full format of an image name is:

```
[registry/][namespace/]image-name[:tag]
```

- **registry**: The container registry (defaults to `docker.io` for Docker Hub)
- **namespace**: The user/organization (defaults to `library` for official images)
- **image-name**: The name of the image
- **tag**: The version (defaults to `latest`)

So `hello-world` is shorthand for `docker.io/library/hello-world:latest`.

You can also use images from other registries. For example:

```sh
docker run ghcr.io/jonashackt/hello-world:latest
```

This pulls the `hello-world` image from GitHub Container Registry (`ghcr.io`) instead of Docker Hub.

## Running Different Types of Containers

Let's explore how different containers behave when you run them.

### 1. Containers That Exit Immediately

The `hello-world` container we just ran prints a welcome message and then exits immediately. This happens because the container's main process completes its task (printing the message) and then terminates.

Try running it again to see:

```sh
docker run hello-world
```

Notice how it prints the message and returns you to your command prompt right away.

### 2. Containers That Run in the Foreground

Now, let's run a web server container using the nginx image from Docker Hub:

```sh
docker run nginx
```

Unlike the `hello-world` container that exited right after printing the welcome message, the Nginx container keeps running in the **foreground**. This means:

- The container stays active and continues running
- Your terminal is "attached" to the container - you can see its logs
- The terminal is blocked until the container stops

To terminate the Nginx container running in the foreground, you can press **Ctrl+C** to send the SIGINT signal to the container. This will stop the container and return control to your terminal.

**Why does Ctrl+C work here?** The nginx process is the main process in the container, and it properly handles the SIGINT signal to shut down gracefully.

### 3. Containers with Interactive Shells

Let's try running a Debian container:

```sh
docker run -it debian
```

The `-it` flags mean:
- `-i`: Keep STDIN open (interactive)
- `-t`: Allocate a pseudo-TTY (terminal)

This gives you an interactive shell inside the container. You'll notice you're now inside the container (your prompt might change).

**Try this:** While inside the container, press **Ctrl+C**. Notice that it doesn't exit the container - it just sends the signal to whatever command you might be running in the shell, but the shell itself keeps running.

**Why doesn't Ctrl+C work here?** The shell (like `bash` or `sh`) intercepts the Ctrl+C signal and forwards it to the current process, but the shell itself continues running. The container only stops when the shell exits.

To exit the interactive shell, you have two options:

1. Type `exit` in the shell itself
2. Press **Ctrl+D** to send the EOF (End of File) signal, which closes the shell

Alternatively, you can stop the container from another terminal using:
```sh
docker stop <container-id>
```

or

```sh
docker kill <container-id>
```

## Why Do These Containers Behave Differently?

You might be wondering: why do `hello-world`, `nginx`, and `debian` behave so differently when we run them with the same `docker run` command? 
The answer lies in what their Dockerfiles specify as the **CMD** or **ENTRYPOINT**.
These are the command that runs when the container starts.

### The hello-world Image

The `hello-world` image's Dockerfile likely has something like:

```dockerfile
CMD ["/hello"]
```

Where `/hello` is a simple program that prints a message and then exits. Once this program finishes, the container's main process terminates, so the container stops.

**Want to see the actual Dockerfile?** Check it out on GitHub: [docker-library/hello-world](https://github.com/docker-library/hello-world)

### The nginx Image

The `nginx` image's Dockerfile likely has:

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

This starts the nginx web server and keeps it running. The `daemon off;` flag prevents nginx from running in the background, so it stays in the foreground as the main process. Since nginx is designed to run continuously (it's a web server), the container keeps running until you stop it.

When you press Ctrl+C, the signal is sent to the nginx process, which handles it gracefully and shuts down, causing the container to stop.

**Want to see the actual Dockerfile?** Check it out on GitHub: [nginxinc/docker-nginx](https://github.com/nginxinc/docker-nginx)

### The debian Image

The `debian` image's Dockerfile likely has:

```dockerfile
CMD ["bash"]
```

or

```dockerfile
CMD ["/bin/bash"]
```

This starts an interactive shell. However, when you run `docker run debian` without the `-it` flags, the shell starts but immediately exits because there's no interactive terminal attached. That's why we use `docker run -it debian`, the `-it` flags provide the interactive terminal that the shell needs.

When you're inside the shell and press Ctrl+C, the shell intercepts the signal. The shell is designed to forward signals to child processes (commands you run inside it), but the shell itself continues running. The container only stops when the shell process exits (via `exit` or Ctrl+D).

**Want to see the actual Dockerfile?** Check it out on GitHub: [docker-library/official-images - debian](https://github.com/docker-library/official-images/tree/master/library/debian)

### Key Takeaway

The behavior of a container depends on what command is specified in its Dockerfile's `CMD` or `ENTRYPOINT`:

- **Short-lived commands** (like `hello-world`): The program runs, completes, and exits → container stops
- **Long-running services** (like `nginx`): The process runs indefinitely → container keeps running until stopped
- **Interactive shells** (like `debian`): The shell waits for input → container keeps running until the shell exits

All three use the same `docker run` command, but their different purposes (demonstration, web server, base OS) result in different behaviors based on what their Dockerfiles define as the main process.

## Running Containers in the Background

So far, we've been running containers in the foreground. But you can also run them in the **background** using the `-d` (detached) flag:

```sh
docker run -d nginx
```

This will:
- Start the container and return immediately to your command prompt
- Run the container in the background
- Print the container ID

The container keeps running even though you're back at your terminal. You can check running containers with:

```sh
docker ps
```

To stop a background container, use:

```sh
docker stop <container-id>
```

**Key Differences:**
- **Foreground** (`docker run nginx`): Terminal is attached, you see logs, Ctrl+C stops it
- **Background** (`docker run -d nginx`): Returns immediately, container runs detached, use `docker stop` to stop it

## Summary

Congratulations! You've learned the fundamentals of running Docker containers. Here's what you now understand:

- How to run containers using `docker run` and why Docker Desktop needs to be running
- The structure of image names and how to use images from different registries
- Why different containers behave differently based on their Dockerfile's `CMD` or `ENTRYPOINT`
- The difference between foreground and background container execution
- How to interact with containers and stop them appropriately

The key insight is that a container's behavior is determined by its main process - whether it's a short-lived program, a long-running service, or an interactive shell. Understanding this will help you work effectively with any Docker container you encounter.

Keep experimenting with different images and flags to solidify these concepts. Try running containers in both foreground and background modes, and explore the Dockerfiles of images you use to understand how they're built!

## Try This

You noticed we ran `docker run -it debian`. What happens if we only run with interactive  flag like `docker run -i debian` or only pseudo terminal flag `docker run -t debian`. 
What is the difference ?