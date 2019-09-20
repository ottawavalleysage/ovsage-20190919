# October Meeting - 2019-09-19

We are continuing the container theme. Tonight we will create a container from
scratch and add a few utilities to it. The good news is that the base image
will be supplied at Dymon so we don't have to kill the bandwidth. The rest
should be small enough that we do not cause issues.

## Firstly though...

Today is international talk like a pirate day, so consider your conatiners to
be treasure chests. We will be using genuine 8 bits of 8 (64 bit) architecture
and we have a nautical theme with Docker itself.

## Building your own container

While Docker Hub provides lots of ready to use containers, they may not always
meet our requirements. Maybe we just do not trust other people's builds, they
don't have what we want, they are too big, etc. This is not a problem, as we
can roll our own for pretty much anything we want.

Tonight, we will use Alpine Linux for our container as it is small - really
small and the project provides a mini system designed around creating
containers.

## Alpine Linux

I've taken the liberty to grab the following from the project page.

### About

Alpine Linux is an independent, non-commercial, general purpose Linux
distribution designed for power users who appreciate security, simplicity and
resource efficiency.

### Small

Alpine Linux is built around musl libc and busybox. This makes it smaller and
more resource efficient than traditional GNU/Linux distributions. A container
requires no more than 8 MB and a minimal installation to disk requires around
130 MB of storage. Not only do you get a fully-fledged Linux environment but a
large selection of packages from the repository.

Binary packages are thinned out and split, giving you even more control over
what you install, which in turn keeps your environment as small and efficient
as possible.

### Simple

Alpine Linux is a very simple distribution that will try to stay out of your
way. It uses its own package manager called apk, the OpenRC init system, script
driven set-ups and that’s it! This provides you with a simple, crystal-clear
Linux environment without all the noise. You can then add on top of that just
the packages you need for your project, so whether it’s building a home PVR, or
an iSCSI storage controller, a wafer-thin mail server container, or a
rock-solid embedded switch, nothing else will get in the way.  

### Secure

Alpine Linux was designed with security in mind. All userland binaries are
compiled as Position Independent Executables (PIE) with stack smashing
protection. These proactive security features prevent exploitation of entire
classes of zero-day and other vulnerabilities.

## Docker and Alpine

Docker has used Apline for a while and it is the default distribution for many
of the examples used. The container from Docker Hub is a little larger than I
want for dedicated tasks, so I'd prefer to build my own. 

We can go over some of the stuff I am planning on using it for either at the
end of the talk or we can make it another talk. Having a suite of containers
for dedicated tasks is fantastic, as it keeps my toolchain independant of OS
updates and changes to my preferred platform. There will be a
[Linux-Ottawa](https://linux-ottawa.org) talk on building a cusomized platform
in November.

## Getting started

Assuming that you still have the configuration from our previous docker events,
we need to build a new `Dockerfile` with the following lines:

```
FROM scratch

ENV ALPINE_ARCH x86_64
ENV ALPINE_VERSION 3.10.2

ADD alpine-minirootfs-${ALPINE_VERSION}-${ALPINE_ARCH}.tar.gz /
CMD ["/bin/sh"]
```

I have a USB stick with the necessary image on it, or you can download it from
the web server running on my MicroPC. I'll provide the address in a moment.

Tonight's address is: 192.168.8.203:8000

Regardless of the system you are using, you will probably want a directory for this exercise. I'm using alpine, but feel free to call it what you want. Perform the following operations:

```
mkdir alpine
cd alpine
curl -O http://192.168.8.203:8000/alpine-minirootfs-3.10.2-x86_64.tar.gz
ls
```

You should see the file and you can now create your `Dockerfile` using the content as shown above.

Once you have created it, issue the build command:

`docker build -t <your-docker-id>/alpine .`

This should go pretty quick and you will have a base image ready to go.

Try it out:

`docker run <your-docker-id>/alpine cat /etc/os-release`

So it is working. Try this one:

`docker run <your-docker-id>/alpine bash`

What happened?

I did mention this was a minimal image and Alpine uses busybox, so the default shell is /bin/sh

Try that: `docker run <your-docker-id>/alpine sh`

No errors right?

Still not quite what you expected? Well, it did what you wanted, just try this:

`docker run -it <your-docker-id>/alpine sh`

The extra flags tell docker that you want a tty allocated and an interactive
session. You get the same result with `docker run --interactive --tty
<your-docker-id>/alpine sh` If you have a ENTRYPOINT defined, you need a little
more on the command line, `docker run -it --entrypoint /bin/sh
<your-docker-id>/alpine`.

Exit from docker

`exit`

## Making this more useful

I suspect that the first thing you would want to do is make sure there are no updates required. Alpine uses yet another package manager called `apk`, so we will utilize that.

Open a shell and update the system:

```
docker run -it <your-docker-id>/alpine sh
apk update
apk upgrade
```

You should get something like this:

```shell
/ # apk update
fetch http://dl-cdn.alpinelinux.org/alpine/v3.10/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.10/community/x86_64/APKINDEX.tar.gz
v3.10.2-58-g4942958220 [http://dl-cdn.alpinelinux.org/alpine/v3.10/main]
v3.10.2-42-g95d37f7648 [http://dl-cdn.alpinelinux.org/alpine/v3.10/community]
OK: 10336 distinct packages available
/ # apk upgrade
(1/2) Upgrading libcrypto1.1 (1.1.1c-r0 -> 1.1.1d-r0)
(2/2) Upgrading libssl1.1 (1.1.1c-r0 -> 1.1.1d-r0)
OK: 6 MiB in 14 packages
/ # 
```

Now Alpine is up to date for the base image. This can be done in the
`Dockerfile` as well. We will go over a few things to do with our image before
we alter the dockerfile.

We might want some software installed. 

**Don't do any of this yet.**

We can do this intereactively as well. What do we want to do with our
container? I might want to have a static site generator installed like the hugo
example we did previously, or maybe a documentation system. We can go with that
tonight. I think I will want Asciidoctor installed, so we should install it.

`apk add asciidoctor`

Do we have curl? Probably not, so get that first:

`apk add curl`

We should have something to test with, so get the test.asciidoc file from the MicroPC 

`curl -O http://192.168.8.203:8000/test.asciidoc`

And since it needs some styling, grab the asciidoc.css as well.

`curl -O http://192.168.8.203:8000/asciidoc.css`

So have a look at the `test.asciidoc` file (I cut out a lot of it here):

```
Asciidoctor Demo
================
////
Big ol' comment

sittin' right 'tween this here title 'n header metadata
////
Dan Allen <thedoc@asciidoctor.org>
:description: A demo of Asciidoctor. This document +
              exercises numerous features of AsciiDoc +
              to test Asciidoctor compliance.
:backend: html5
:library: Asciidoctor
:stylesheet: asciidoc.css
:idprefix:
//:doctype: book
//:sectids!:
// the previous three attributes customize the generated output

[role='lead']
This is a demonstration of {library}. And this is the preamble of this document.

[[purpose]]
.Purpose
****
This document exercises many of the features of AsciiDoc to test the Asciidoctor implementation.
****

TIP: If you want the output to look familiar, copy (or link) the AsciiDoc stylesheet, asciidoc.css, to the output directory.

NOTE: Items marked with TODO are either not yet supported or work in progress.

```

So we should convert it to HTML with Asciidoctor:

`asciidoctor test.asciidoc`

You will end up with a HTML file that looks pretty good (with the css included).

Of course, viewing it will be difficult, we need a web server.

Install lighttpd and create a minimal config for it.

`apk add lighttpd`

We need a place to serve web pages from, so for now create a /www directory

```
mkdir /www
mkdir /www/pages
```

create a minimal config file for lighttpd:

```
server.document-root = "/www/pages/" 

server.port = 3000

mimetype.assign = (
  ".html" => "text/html", 
  ".txt" => "text/plain",
  ".jpg" => "image/jpeg",
  ".png" => "image/png" 
)
```

Rather than have to do this a couple of times, we can adjust our `Dockerfile` now.

Rename your `Dockerfile` to `Dockerfile-alpine` and create a new one.

**Note:** This is not completed yet... currently in edit

```
FROM <your-docker-id>/alpine

ENV DOCUMENT_DIR=/www

RUN apk update && apk upgrade \
    && apk add asciidoctor lighttpd curl

RUN gem install --no-document asciidoctor-pdf --pre
RUN gem install --no-document rouge asciidoctor-diagram coderay

RUN mkdir ${DOCUMENT_DIR}

WORKDIR ${DOCUMENT_DIR}

VOLUME ${DOCUMENT_DIR}

CMD ["lighttpd", "-D", "-f","lighttpd.conf"]
```

So we will rebuild our image, or rather create a new image based on our existing base image:

`docker build -t <your-docker-id>/alpine-asciidoctor .`

This will take a little longer. When completed, run it to get a command shell

`docker run -it <your-docker-id>/alpine-asciidoctor sh`

When you get your prompt, you can check to see if you can see the installed software:

```
which asciidoctor
which lighttpd
which curl
```

You also whould have started up in the /www directory

# References and Useful Info

- [Alpine Linux](https://alpinelinux.org)
- [Asciidoctor](https://asciidoctor.org)
- [Lighttpd](http://lighttpd.org)

## Getting started with Docker

From Grigor Khachatryan's Docker for Beginners series:
- [Part 1: Setup](https://medium.com/@Grigorkh/docker-for-beginners-part-1-setup-e3ad9f4ba25e)
- [Running your first container](https://medium.com/@Grigorkh/docker-for-beginners-part-2-running-your-first-container-7cb1ef829f79)
- [Webapps with Docker](https://medium.com/@Grigorkh/docker-for-beginners-part-3-webapps-with-docker-18f2243c144e)
- [Deploying an app to a Swarm](https://medium.com/@Grigorkh/docker-for-beginners-part-4-deploying-an-app-to-a-swarm-620b4d67e7c3)
