---
title: Dockerfile 命令笔记
date: 2020-11-09 15:14:39
---

<!--more-->

## [FROM](https://docs.docker.com/engine/reference/builder/#from)

```dockerfile
FROM [--platform=<platform>] <image> [AS <name>]
#or
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
#or
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```

+ `ARG` is the only instruction that may precede `FROM` in the `Dockerfile`.
+ The optional `--platform` flag can be used to specify the platform of the image in case `FROM` references a multi-platform image. For example, `linux/amd64`, `linux/arm64`, or `windows/amd64`. By default, the target platform of the build request is used.
+ The `tag` or `digest` values are optional. If you omit either of them, the builder assumes a `latest` tag by default. The builder returns an error if it cannot find the `tag` value.
+ Optionally a name can be given to a new build stage by adding `AS name` to the `FROM` instruction. The name can be used in subsequent `FROM` and `COPY --from=<name>` instructions to refer to the image built in this stage.

<!--more-->

## [ARG](https://docs.docker.com/engine/reference/builder/#arg)

```dockerfile
ARG <name>[=<default value>]
```

The `ARG` instruction defines a variable that users can **pass at build-time** to the builder with the `docker build` command **using the `--build-arg <varname>=<value>` flag**. 

+ If an `ARG` instruction has a default value and if there is no value passed at build-time, the builder uses the default.

## [ENV](https://docs.docker.com/engine/reference/builder/#env)

```dockerfile
ENV <key>=<value> ...
```

The `ENV` instruction sets the environment variable `<key>` to the value `<value>`. This value will be in the environment for all subsequent instructions in the build stage and can be [replaced inline](https://docs.docker.com/engine/reference/builder/#environment-replacement) in many as well. The value will be interpreted for other environment variables, so quote characters will be removed if they are not escaped. Like command line parsing, quotes and backslashes can be used to include spaces within values.

The environment variables set using `ENV` will persist when a container is run from the resulting image. You can view the values using `docker inspect`, and **change them using `docker run --env <key>=<value>`.**

## [RUN](https://docs.docker.com/engine/reference/builder/#run)

```dockerfile
RUN <command>
#or
RUN ["executable","param1","param2"]
```

+ `RUN <command>` (*shell* form, the command is run in a shell, which by default is `/bin/sh -c` on Linux or `cmd /S /C` on Windows)
+ `RUN ["executable", "param1", "param2"]` (*exec* form)

## [SHELL](https://docs.docker.com/engine/reference/builder/#shell)

```dockerfile
SHELL ["executable","parameters"]
```

The `SHELL` instruction allows the default shell used for the *shell* form of commands to be overridden. The default shell on Linux is `["/bin/sh", "-c"]`, and on Windows is `["cmd", "/S", "/C"]`. The `SHELL` instruction *must* be written in JSON form in a Dockerfile.

The following instructions can be **affected** by the `SHELL` instruction when the *shell* form of them is used in a Dockerfile: `RUN`, `CMD` and `ENTRYPOINT`.

**[What is the difference between using bash and sh to run a script?](https://unix.stackexchange.com/questions/270966/what-is-the-difference-between-using-bash-and-sh-to-run-a-script)**

## [CMD](https://docs.docker.com/engine/reference/builder/#cmd)

```dockerfile
CMD ["executable","param1","param2"]
#or
CMD ["param1","param2"]
#or
CMD command param1 param2
```

- `CMD ["executable","param1","param2"]` (*exec* form, this is the preferred form)
- `CMD ["param1","param2"]` (as *default parameters to ENTRYPOINT*)
- `CMD command param1 param2` (*shell* form)

There can only be one `CMD` instruction in a `Dockerfile`. If you list more than one `CMD` then only the last `CMD` will take effect.

**The main purpose of a `CMD` is to provide defaults for an executing container.** These defaults can include an executable, or they can omit the executable, in which case you must specify an `ENTRYPOINT` instruction as well.

**If the user specifies arguments to `docker run` then they will override the default specified in `CMD`**

## [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint)

```dockerfile
ENTRYPOINT ["executable","param1","param2"]
#or
ENTRYPOINT command param1 param2
```

An `ENTRYPOINT` allows you to configure a container that will run as an executable.

Command line arguments to `docker run <image>` will be **appended** after all elements in an *exec* form `ENTRYPOINT`, and will override all elements specified using `CMD`. This allows arguments to be passed to the entry point, i.e., `docker run <image> -d` will pass the `-d` argument to the entry point. You can override the `ENTRYPOINT` instruction using the `docker run --entrypoint` flag.

**[Understand how CMD and ENTRYPOINT interact](https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact)**

## [LABEL](https://docs.docker.com/engine/reference/builder/#label)

```dockerfile
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

The `LABEL` instruction adds metadata to an image. A `LABEL` is a key-value pair. To include spaces within a `LABEL` value, use quotes and backslashes as you would in command-line parsing. 

## [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose)

```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

The `EXPOSE` instruction informs Docker that the container listens on the specified network ports at runtime. You can specify whether the port listens on TCP or UDP, and the default is TCP if the protocol is not specified.

The `EXPOSE` instruction does not actually publish the port. It functions as a type of documentation between the person who builds the image and the person who runs the container, about which ports are intended to be published. To actually publish the port when running the container, use the `-p` flag on `docker run` to publish and map one or more ports, or the `-P` flag to publish all exposed ports and map them to high-order ports.

## [ADD](https://docs.docker.com/engine/reference/builder/#add)

```dockerfile
ADD [--chown=<user>:<group>] <src>... <dest>
#or
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

The `ADD` instruction copies new files, directories or remote file URLs from `<src>` and adds them to the filesystem of the image at the path `<dest>`.

The `<dest>` is an **absolute path**, or a path **relative to `WORKDIR`**, into which the source will be copied inside the destination container.

The `<src>` path must be inside the *context* of the build.

## [COPY](https://docs.docker.com/engine/reference/builder/#copy)

```dockerfile
COPY [--chown=<user>:<group>] <src>... <dest>
#or
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

The `COPY` instruction copies new files or directories from `<src>` and adds them to the filesystem of the container at the path `<dest>`

The `<src>` path must be inside the *context* of the build.

## [VOLUME](https://docs.docker.com/engine/reference/builder/#volume)

```dockerfile
VOLUME ["/data"]
```

The `VOLUME` instruction creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers. The value can be a JSON array, `VOLUME ["/var/log/"]`, or a plain string with multiple arguments, such as `VOLUME /var/log` or `VOLUME /var/log /var/db`. 

## [USER](https://docs.docker.com/engine/reference/builder/#user)

```dockerfile
USER <user>[:<group>]
#or
USER <UID>[:<GID>]
```

The `USER` instruction sets the user name (or UID) and optionally the user group (or GID) to use when running the image and for any `RUN`, `CMD` and `ENTRYPOINT` instructions that follow it in the `Dockerfile`

## [WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir)

```dockerfile
WORKDIR /path/to/workdir
```

The `WORKDIR` instruction sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the `Dockerfile`. If the `WORKDIR` doesn’t exist, it will be created even if it’s not used in any subsequent `Dockerfile` instruction.

The `WORKDIR` instruction can be used multiple times in a `Dockerfile`. If a relative path is provided, it will be relative to the path of the previous `WORKDIR` instruction.

## [STOPSIGNAL](https://docs.docker.com/engine/reference/builder/#stopsignal)

```dockerfile
STOPSIGNAL signal
```

The `STOPSIGNAL` instruction sets the system call signal that will be sent to the container to exit. This signal can be a valid unsigned number that matches a position in the kernel’s syscall table, for instance 9, or a signal name in the format SIGNAME, for instance SIGKILL.

## [HEALTHCHECK](https://docs.docker.com/engine/reference/builder/#healthcheck)

```dockerfile
HEALTHCHECK [OPTIONS] CMD command
#or
HEALTHCHECK NONE
```

- `HEALTHCHECK [OPTIONS] CMD command` (check container health by running a command inside the container)
- `HEALTHCHECK NONE` (disable any healthcheck inherited from the base image)

The `HEALTHCHECK` instruction tells Docker how to test a container to check that it is still working. This can detect cases such as a web server that is stuck in an infinite loop and unable to handle new connections, even though the server process is still running.

## References

+ https://docs.docker.com/engine/reference/builder/
