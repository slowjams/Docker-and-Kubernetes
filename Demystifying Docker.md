## Demystifying Docker

## What's a Docker Image

`docker run image` is a combination of `docker create` and `docker start`

```js
docker create Busybox // create a container (say id is abc123) whose status is "Created"
docker start abc123   // start the container the status is "running" and then "exit" (status might always be running depends on the nature of the imag
```

An image contains a small filesytem (e.g container home, bin, proc, root, et) and a default startup command. When do `docker create`, this image gets copied to your hard disk, that's all.

When you run `docker start`, the startup commands run so we have a running process.

Let's say you do `docker run Busybox echo hi there` where `echo hi there` is the command to overide the default command (the default command is not important to us). So the container run and print "hi there" then exit, and this new command is baked into the container. So if you start container again, it will print "hi there" again and exit


## Stop/Kill an Container

```js
docker stop  // send a SIGTERM signal, which allows you do a little bit clean-up (e.g flush buffer to save file etc) in your codebase when listen to this signal

docker kill  // send a SGKILL signal, which is like you have to shut down and you are not allowed to do any additional work
```

when you do `docker stop` and the container is not stoppped for 10 sec, the docker will automatically issue `docker kill`.


## Execute an Command on a Container

```js
// say you already have redis container running and want to use its cli
docker exec -it abc1234 redis-cli

docker exec -it abc1234 sh // run shell command. Container already runs before docker exec, so the default command has been run, if you don't default command to run do the follwing

docker run -it redis sh // overwrite the default command so no other process will be running except shell, can be useful in some scenario
```

Every process has three data stream i.e `STDIN` / `STDOUT` / `STDERR`. When a process is running in a container, by default terminal is connected with `STDOUT` stream of the process running get `STDIN` stream added when `i` is used. `-t` is used for formatted input operations (need to learn Linux hard to know what's tty)

so what does `-t` do? This is also best illustrated by example:

`docker run -e MYSQL_ROOT_PASSWORD=123 -i mariadb mysql -u root -p` will give you a password prompt. If you type the password, the characters are printed visibly.

`docker run -i alpine sh` will give you an empty line. If you type a command like `ls` you get an output, but you will not get a prompt or colored output.

In the last two cases, you get this behavior because mysql as well as shell were not treating the input as a tty and thus did not use tty specific behavior like asking the input or coloring the output.


## Docker File

```js
FROM alpine  // Step 1

RUN apk add --update redis   // Step 2

CMD["redis-server"]  // Step 3
```

note that each step creates an intermittent running container then this container is removed and its filesystem is used to create a new image for next step.
for example, after Step 1, an intermittent image is created, and at Step 2, an intermittent container is created by the image at Step 1, and the container runs `apk add --update redis`, which adds a redis related files in the container's filiesystem which is used to create an imtermittent image for the next instruction and this intemittent container is removed too. So the image created at Step 3 is the final image to be used

```js
docker commit -c "CMD 'redis-server'" containerId  // build a new image based on container (container's filesystem will be used to create the image)

docker build --progress=plain .
```

```js
FROM node:14-alpine

COPY ./ ./

RUN npm config set strict-ssl=false
RUN npm install

CMD["npm", "start"]
```

=================================================================================================================================================================


## Docker-Compose

`docker run myImage` = `docker-compose up`

`docker build .` + `docker run myImage` = `docker-compose up --build`

note that by default, `docker-compose up` creates default network for its container to communicate with each others

```yml
version: '3'

services:
  redis-server:
    image: 'redis'
  node-app:
    restart: always
    #restart: "no"            // default, note that no has to be quoted because yml interpret unquoted no as false, which is different to "no" string
    #restart: always          // restart container for any reason such as exit(0) and exit(1,2, ...). If it's manually stopped, it's restated only when Docker daemon restarts 
                              # or the container itself is manually restarted
    #restart: on-failure      // restart for exit(1,2, ...) only
    #restart: unless-stopped  // similar to always, except that when the container is stopped, it isn't restarted even after Docker daemon restarts
    build: .
    ports:
      -"4001:8081"
```


