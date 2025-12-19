As we discussed earlier, a container's default command is specified in its Dockerfile's `CMD` instruction. The default behavior of `docker run nginx` is driven by what is specified in that Dockerfile.

**Want to see an example?** Check out the nginx Dockerfile: [nginx/docker-nginx](https://github.com/nginx/docker-nginx/blob/afa829ae8cd9e25cf539cb03167dff1162f852cb/mainline/debian/Dockerfile)

But what if we want different behavior? We can override the default commands!

## Overriding Default Commands

Let's explore this concept using the Redis image as an example.

First, let's run Redis in the default way:

```sh
docker run redis
```

By default, when we run this command, the Redis image runs the `redis-server` executable binary. However, the Redis image also contains the `redis-cli` (Redis client) executable binary, which we might want to use.

**What just happened?**

The container started and is running `redis-server` in the foreground (or in detached mode if you used `-d`). The `redis-server` is the default command specified in the Redis image's Dockerfile.

## Executing Commands in Running Containers

In order to interact with our `redis-server` running in a container, we need to know the container's unique identifier so we can execute commands inside it.

First, let's list the running containers to get the container ID:

```sh
docker ps
```

From there, you can get the Redis container ID (or name).

Now that we know the container is running and we have its ID, there should be a way to execute commands inside running containers from outside.

### The `docker exec` Command

`docker exec` is a command that lets us execute instructions inside a running container from outside. The syntax is:

```sh
docker exec <container_id> <command>
```

We're telling Docker to **EXEC**ute a command into the specified **CONTAINER_ID**.

For example, to run `redis-cli` inside our Redis container:

```sh
docker exec <container_id> redis-cli
```

**Try this:** Run the command above. You might notice that nothing obvious happened or the command returned immediately.

**Why is this happening?**

The `redis-cli` command actually ran, but nothing obvious appeared in your terminal outside the container. This is because we're not running in interactive mode. The command executed, but there's no way to interact with it We can't see its output properly or send input to it.

## Running Commands Interactively

To properly interact with commands executed inside a container, we need to use the `-it` flags with `docker exec`, just like we did with `docker run`:

```sh
docker exec -it <container_id> redis-cli
```

The `-it` flags mean:
- `-i`: Keep STDIN open (interactive)
- `-t`: Allocate a pseudo-TTY (terminal)

This asks Docker to make the container receive input from the terminal (`-i`) as well as send output to the terminal (`-t`).

**Now you'll see:** You're inside the Redis CLI! Try running `ping` and you should get a `PONG` response from the server, confirming that you're successfully communicating with the Redis server running in the container.

You can now interact with Redis:
- Type `set mykey "Hello World"` to set a value
- Type `get mykey` to retrieve it
- Type `exit` or press **Ctrl+D** to exit the Redis CLI

## Understanding `docker run` vs `docker exec`

It's important to understand the difference between these two commands:

| Command | Purpose | When to Use |
|---------|---------|-------------|
| `docker run` | Creates and starts a new container | When you want to start a new container instance |
| `docker exec` | Executes a command in an existing running container | When you want to run additional commands in a container that's already running |

**Key differences:**
- `docker run` creates a new container and runs the default command (or a command you specify)
- `docker exec` runs an additional command in an already-running container
- You can use `docker exec` multiple times on the same container to run different commands
- `docker exec` requires the container to be running (use `docker ps` to check)

## Overriding Commands at Container Start

You can also override the default command when starting a container by appending a command to `docker run`:

```sh
docker run redis redis-cli --help
```

This starts a new Redis container but runs `redis-cli --help` instead of the default `redis-server` command. The container will start, execute the command, and then exit (since `redis-cli --help` just prints help and exits).

**Note:** When you override the command like this, the container runs that command instead of the default one. If the command exits, the container stops.

## Summary

Congratulations! You've learned how to execute commands in running containers and override default commands. Here's what you now understand:

- How container default commands are specified in Dockerfiles via the `CMD` instruction
- How to override default commands when starting containers
- How to use `docker exec` to execute commands in already-running containers
- The importance of the `-it` flags for interactive commands
- The difference between `docker run` (creates new container) and `docker exec` (executes in existing container)
- How to interact with services running inside containers (like Redis CLI with Redis server)

The key insight is that `docker exec` allows you to run additional processes inside a running container, which is essential for debugging, monitoring, and interacting with services. Understanding when to use `docker run` versus `docker exec` will help you work effectively with Docker containers.

Keep experimenting with different commands and containers to solidify these concepts!

## Try This

1. Start a Redis container in detached mode: `docker run -d redis`
2. Get the container ID using `docker ps`
3. Try running `docker exec <container_id> redis-cli` without `-it` - what happens?
4. Now run `docker exec -it <container_id> redis-cli` - what's the difference?
5. While in the Redis CLI, try:
   - `ping` (should return PONG)
   - `set test "Hello Docker"`
   - `get test`
   - `exit`
6. Try running `docker exec -it <container_id> sh` to get a shell inside the container. What can you do there?
7. **Food for thought:** Now that you know how to run `redis-cli` in a container, how can you talk with a Redis server that is running in a different container or on a different machine?
