## Docker 

While building your .dockerfile, there are several important concepts to keep in mind that will ensure proper performance of your container, and expressing your true intentions for building the container in the first place

1) CMD vs. ENTRYPOINT

> THERE CAN ONLY BE ONE <B>CMD</B> AND ONE <B>ENTRYPOINT</B> IN A .dockerfile

RUN, CMD, and ENTRYPOINT can all be used to run commands within our container.  So what makes CMD and ENTRYPOINT unique?

One way to understand what sets these two commands apart from RUN is to think about whether you wish to run a container as an `executable`.  What does running a container as an executable mean?  It means that your container is intended to run a primary script/process and that's about it.  A simple example of this is running a server built on top of a base Nodejs image.

```docker
#.dockerfile to create my server image
FROM node AS base
...
ENTRYPOINT ["node", "index.js"] #run this process
CMD ["flag 1", "flag 2"] #provide some default args for the process
```

We can build the representative .dockerfile above into an image tagged as `server`

>$docker build -t server .

If we spin up a container from the `server` image, and provide no other arguments following our image name, the ENTRYPOINT and CMD flags take over.

>$docker run --name server -p 9000:9000 -v my_volume server

If we <i>do</i> provide args following the image name, those args will supercede our CMD, but NOT the entrypoint command.
```sh
#arg 1 and arg2 supercede flag1 and flag2 specified in the .dockerfile CMD above

docker run --name server -p 9000:9000 -v my_volume server arg1 arg2
```
This design pattern makes clear my intentions that this container is serving a very specific purpose by limiting the API facing end-users to ensure that index.js is run as PID1.

Understand that I could have written the .dockerfile using only the `CMD` without `ENTRYPOINT`.

```docker
FROM node AS base
...
CMD ["node", "index.js", "flag 1", "flag 2"] #provide some default args for the process
```

The potential downside to this approach is that user args provided to the container runtime would not only supercede flag 1 and flag 2, but the call to execute my server!

## Shell Form vs Exec Form
RUN, CMD, and ENTRYPOINT can all be specified in one of two forms:

1)  <b>Shell form</b>
    ```docker
    RUN echo hello
    ```

2)  <b>Exec form</b>
    ```docker
    RUN ["/bin/bash", "echo", "hello"]
    ```

>"When using the shell form, the specified command is executed with a /bin/sh -c.  
>When you use docker stop or docker kill to signal a container, that signal is sent only to the container process running as PID 1.
>Since /bin/sh doesn't forward signals to any child processes, the SIGTERM we sent never reached our script. Clearly, if we want our app to be able to receive signals from the host we need to find a way to run it as PID 1.  [[1](https://www.ctl.io/developers/blog/post/gracefully-stopping-docker-containers/)]

So the unintended side effect of using `Shell` form is that our processes will be run inside of a shell process that is unable to forward signals it is receiving from the host machine.  

`Exec` form commands do not invoke a shell to run in, meaning that your ENTRYPOINT is serving as PID1 and is directly able to receive signals from the host machine.





