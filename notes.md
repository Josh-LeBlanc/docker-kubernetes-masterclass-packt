---
id: Docker and Kubernetes Masterclass
aliases: []
tags:
  - docker
  - kubernetes
---
# intro
ngl this looks pretty good i think
part 1 is docker, part 2 is k8s
# part 1, docker
# ch 4. running containers
## running our first container: nginx - lab

1. find image on docker hub, it gives you a command with the name `docker pull nginx`
2. run the image, i assume this creates a container: `docker run nginx`
    `docker ps` shows our running container
### run in background
`docker run -d nginx` runs it in the background. make sure to put flags before name
    `docker ps` shows our running container
    `docker kill <name>` will kill the container, with the name listed in docker ps
### map port and give name
`docker run -d -p 8080:80 --name web-server nginx`
- this maps our local port 8080 to port 80 on the container, forwarding all requests to 8080 for us to the container port 80
- it also gives it a selected name
### killing
`docker stop` is "more graceful" than `docker kill`
## container lifecycle
### running
`docker run <image>` = `docker create <image>?` + `docker start <cId>?` double check this
then the container is now running, we can do things like `docker logs`, `docker inspect`, `docker exec`
### pause
`docker pause <cId>` pauses the container, maintains the memory in the state when it was paused

unpause it with `docker unpause <cId>`
### stopping
`docker stop <cId>` stops the container gracefully and clears memory
`docker kill <cId>` stops the container forcefully with SIGKILL
#### container exit code
0 means no error
nonzero means error
### container stopped state
stopped containers can be viewed with `docker ps -a`

you can still `docker logs` and `docker inspect` in this state
### running stopped container
`docker start <cId>` will make a stopped container start running again

`docker rm <cId>` removes the container from the system
## docker CLI - lab
### removing an image (not a container)
`docker rmi <image id>`. chatGPT tells me to use the image ID, but I used the name nginx and it worked fine.
### step one, pull down ubuntu image
`docker pull ubuntu`
### run it
`docker run ubuntu`
it exits immedately, because there is nothing keeping it alive

as opposed to nginx where it stays up and waits for connections
### run nginx twice
`docker run nginx` stop it, then run again

it creates a new container each time you docker run, because `run` is a synonym of create and start.

to run a container that has been stopped, we use `docker start <container id>`, i think you can use the container name as well
### looking for a certain image
`docker ps --filter name=<name>`
### stopping all running containers
`docker stop $(docker ps -q)` docker ps -q just returns all the container ids, so this is very nice. change the flag to -aq to get stopped containers
### viewing logs
`docker logs <container name>`
not sure if this is just for when a container is running in the background but that's what we're doing in this example

add the -f flag if you would like to stay in the logs and see them printed to terminal as they come in
### interactive shell with container
`docker exec -it <container name> /bin/bash`
### docker build
this requires a dockerfile
```dockerfile
FROM ubuntu:latest

CMD ["echo", "Hello from my first docker image!"]
```
then we run `docker build .` if its in the directory, or i assume you can just specify the Dockerfile

this will create an image locally

if you do `docker run <img id>` it will print "hello from my first docker image"
## docker run --help
just a help flag like usual
# ch 5. project: customizing nginx pages
## running nginx container
pulling nginx 1.27.0

run it in detached mode, map port 80 to port 80, name it web server
`docker run -d -p 80:80 --name web-server nginx:1.27.0`

then we get an interactive terminal into the container with
`docker exec -it web-server /bin/bash`

then we install vim with `apt update` and `apt install vim`

now we can edit the file at `/usr/share/nginx/html/index.html`

and then if we look at it with broswer its different woohoo!!
# ch 6. introduction to docker images
## what are docker images
docker images are the _dna of our containers_. they should contain everything needed to run an application

layers:
- __base layer__: an operating system like alpine or ubuntu
- __runtime environment__: python, nodejs
- __libraries and dependencies__: external dependencies, 
- __application code__: source code or compiled binary
- __configuration__: settings for application and its environment

images can be found in the docker hub, private image repositories, or you can build your own!
## working with container registries
registries offer many benefits to your container development:
collaboration, versioning, security, automation (CI/CD)

types of registries: public and private

what to consider when selecting registries:
hosting type, security features, integrations, cost model
## exploring docker hub - lab
nothing of note
## logging into docker hub from docker cli - lab
created a dockerhub account and ran `docker login` and we're in

now we can `docker search <image name>`
## managing docker images with the docker cli - lab
building our first image (with a name):
`docker build -t simple_hello_world .` you must specify a path with a Dockerfile in it, not the dockerfile itself

tag that image with a different version:
`docker tag my_first_image:latest j0shle/my_first_image:v0.0.1`

and then we can push that image to dockerhub under our user:
`docker push j0shle/my_first_image:v0.0.1`

quite cool, but deleting now since it's useless
## introduction to dockerfiles
defines steps for creation of docker images

as each instruction executes a new image is created, we've heard about this _layered_ idea
## dockerfile for nginx
writing a dockerfile to do all of the things we did in the nginx custom file lab

it installs the version of nginx we want, installs vim, and copies over our index.html from our computer

he is getting a 403 forbidden when curling the nginx server because the file is not owned by the nginx user or group, we don't get that error but I will put the line in for future reference anyways 
## understanding the differences between containers and images - lab
i think i have a pretty firm grasp on this

yeah
# ch 7. conatinerize an express application
backend api with nodeJS and express

containerize and run it
## hello world express app
`npm init` to create a (node project)?

the install packages `npm install express@4.19.2 body-parser@1.20.2 --save-exact`

then we wrote a little javascript to start an express application, handle a get request to '/' and just print hello world, and listen on port 3000.

then in package.json `scripts` section we write a script called "start" that maps to "node index.js"

then we can do `npm start` and it goes up, and we can curl localhost:3000 and it says hello world!
## saving and returning users
just a post route to add a user to the user list and a get route to get all the users
## testing app
with postman
## dockerizing our express app
base image is node:22-alpine

setting `WORKDIR /app` which means all following commands will be executed in the /app directory of the container

the we copy over `package.json` and `package-lock.json`

then we install dependencies by RUNning `npm ci`

then we copy over source code

then we expose port 3000

then we start with application with `CMD [ "node", "index.js" ]` which i believe could also be `CMD [ "npm", "start" ]` 

then we build it, give it a version
`docker build -t express-app:v1.0.0 .`

then we run it, map port 3000 to 3000
`docker run -d -p 3000:3000 --name express-app express-app:v1.0.0`
## exploring container interactions and cleanup
when you stop the container it loses all data, that makes sense

when you pause a container, it is still "running", it is just paused

when you resume a paused container, it still holds all the data
# ch 8. deep dive into images
## understanding docker layered architecture - lab
each instruction in the dockerfile creates a new layer in the image, which is a new image each time

`docker history <image name>` shows the layers of the image as it's built
## exploring build contexts - lab
`docker build -t hello-from-node .` the '.' is the build context, meaning all of the files that are passed to the docker daemon when we are building the image

thats pretty much it
## using dockerignore to skip files - lab
docker ignore is the same as git ignore why is this video 8 minutes long i hate frontend people

nvm something cool:
run an instance of a container that is removed after you are done with it (NOT EVEN STOPPED, this is good), and override the CMD with `sh`
`docker run --rm -it hello-from-node sh`
## introduction to environment variables - lab
simple express app:
```bash
npm init -y
npm i express@4.19.2
```

it is just an instruction in the dockerfile: `ENV PORT=3000`

in javascript you get environment variables like `const port = process.env.PORT` (he's making it sound like it's node specific so maybe not all javascript)
### sidenote: cleanup wisdom from claude
view storage taken up by docker with `docker system df`

`docker system prune` to remove stopped containers, unused networks, dangling images, and build cache
`docker system prune -a` to also remove unused images
`docker system prune -a --volumes` to include volume cleanup
## setting environment variables via the CLI
when running the container, just pass them in with -e: `docker run -e PORT=3000` or for what i was wondering: `docker run -e API_KEY=$API_KEY -e API_KEY2=$APIKEY2` or `--env-file .env`
## .env file, shown above
## CMD vs. ENTRYPOINT
CMD can be overwritten by typing a command after the image name at the end of the run command

ENTRYPOINT will just append the commands to the entrypoint command, can be overwritten with --entrypoint flag (after image name) and specifying the command
    or you can specify --entrypoint before the image name, and more commands after the image name `--entrypoint "echo" <image name> "hello world"`

common practice to specify the command with ENTRYPOINT, and the arguments that could be overwritten with CMD
### specify non-default dockerfile:
`docker build -t <tagename> -f <dockerfile name> .`
## distroless images
__distroless images__ are minimal images that contain only the dependencies needed to run our application - no operating system, other binaries, etc
## introduction to multistager dockerfiles - lab
lab in `multistage-builds/`
being able to have multiple FROM instructions - you can have a _build_ phase and a _run_ phase

making a simple express app again

first new things is getting a distroless image from google container registry
rather than using node-alpine, we are using `FROM gcr.io/distroless/nodejs22`

when we do it this way, we cannot install the dependencies because the distroless image does not have `npm` installed
## multistage dockerfiles and distroless images in practice - lab
continued from above

ok we start our build stage with `FROM node:22-alpine AS build`
set our WORKDIR, COPY package* ., RUN npm ci

THEN we get our distroless image,
set WORKDIR /app, `COPY --from=build /app/node_modules node_modules`, COPY src src, ENV var, and our `CMD ["src/index.js"]` because the entrypoint on the distroless image is `node`
## adding typescript to the project - lab
continued from above

install typescript with the relevant types
`npm i --save-dev --save-exact typescript@5.5.3 @types/express@4.17.21`

initialize typescript project
`npx tsc --init`

changed `src/index.js` to `src/index.ts`, changed the `require` statement to an `import` statement

set the `outDir` to "dist" in the `tsconfig.json` file

made a 'build' script in the package.json that just runs `tsc` - which refers to the `tsc` binary in `node_modules`
    which allows us to transpile .ts files into .js files in the dist dir
## updating dockerfile to transpile typescript
build stage:
1. copy the src to the container in the build stage rather than the run stage (after RUN npm ci)
2. copy the tsconfig.json file
3. `RUN npm run build`
run stage:
4. copy dist directory with the transpiled javascript from the build stage (instead of copying src from computer)
5. change CMD to dist/index.js instead of src/index.js
## optimizing docker images - base images - lab
working in `optimizing-images/`

we are installing express as a runtime dependency and typescript and jest as development dependencies

pretty simple, he showed the slim and alpine versions of the node base image that are smaller
## optimizing docker images - instruction order - lab
rather than copying the source code over after building the depenedencies, we copy everything over at the same time.

this means it will not be able to cache the dependency build layer because it will change every time we change the source code, causing longer build times
## optimizing docker images - managing dependencies - lab
install all the dependencies necessary for building in the build phase,

in the run phase, install only the dependencies necessary for running with RUN npm ci --only-production
## optimizing our typescript dockerfile
created a new stage called deps, where we use the same base image as build, same WORKDIR and same COPY package*, but we `RUN npm ci --only-production` so that we only have runtime dependencies
# ch 9. project - containerize a react app
write a react app, build with multistage docker file, have docker file serve static files through nginx server
## setting up react app
set up a react app with create-react-app with typescript template, create dir containerize-react-app
`npx create-react-app --template typescript containerize-react-app`

right from there, you can test it with `npm start`, which spins up a development server
## building and serving the react app
to create an optimized, production build we do
`npm run build`

this will create a build folder, which we can server with
`npx http-server@14.1.1 build`

note:
this doesn't hot reload, we have to run npm run build again which makes sense
## creating a dockerfile for our react app
build stage with node:alpine where we copy over package*, run npm ci to install deps, copy over src/, public/ and tsconfig.json, then run npm run build script

run stage with nginx, we just copy from build stage /app/build /usr/share/nginx/html

and that's literally it you don't need a CMD or ENTRYPOINT in nginx container
# ch 10. volumes and data persistence
## understanding the need for volumes - lab
starting by just running an nginx container for persistence through this lesson

ok basically volumes are a place to keep data when you need it in multiple containers
## introduction to docker volumes
persist data beyond containers lifecycle and between containers

__type of volumes__:
- __bind mounts__: directly link host system files or directories to the container
- __named volumes__: created via docker cli and reusable across containers
## working with bind mounts - lab
alright we are making a Dockerfile.dev in the react app project where instead of `npm run build` and then copying over to a run stage, we just do `npm start`

although npm start supports hotreloading, the files we edit on host aren't going into the container, so it is not updated.

to fix this, we will mount __bind mounts__ to the container, at container run time:
`docker run --rm -d -p 3000:3000 -v ./public:/app -v ./src:/app react-app:dev`
p.s. make sure to do ./<dir> to add volumes from host otherwise it doesn't work
## using named volumes - lab
create a volume
`docker volume create website-data`

now you can mount it to multiple containers:
`docker run -d -p 3000:80 --name website-main -v website-data:/usr/share/nginx/html nginx:1.27.0`
`docker run -d -p 3001:80 --name website-readonly -v website-data:/usr/share/nginx/html nginx:1.27.0`

one thing i noticed:
- when the volume was empty, it we overridden with the files that were generated by the nginx image in the html dir.
- however, once we put stuff into the volume ourselves (echoing hello world > index.html), creating a new nginx container with the volume does not override
    - why does this happen?
## managing docker volumes with the CLI
show all volumes:
`docker volumes ls`
    use -f for filter
    `docker volumes ls -f name=<name> dangling=true`

show details about a volume:
`docker volume inspect <volume name>`

remove a volume:
`docker volume rm <volume name>`
# ch 11. advanced docker topics - resource limits, restart policies, networking

