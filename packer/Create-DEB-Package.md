# Creating and hosting your own deb packages and apt repo

39 minute read Updated:  July 19, 2023

Alex Couture-Beil

As an Ubuntu user, I find myself typing  `apt install ...`  frequently as a way to install software on my system. But what if I wanted to distribute my code to others via an apt repository? In this post I’ll cover how to 1) create a deb package, 2) create an apt repo, 3) signing that apt repo with a PGP key, and 4) putting it all together with some tests.

## Prerequisites

This tutorial assumes you are using Ubuntu, and that the following packages are installed:

```
sudo apt-get install -y gcc dpkg-dev gpg
```

## Step 0: Creating a Simple Hello World Program

Before getting started with packaging, let’s create a basic hello world program under  `~/example/hello-world-program`. To do this, you can copy and paste the following commands into your terminal:

```
mkdir -p ~/example/hello-world-program

echo '#include <stdio.h>
int main() {
    printf("hello packaged world\n");
    return 0;
}' > ~/example/hello-world-program/hello.c
```

Then, you can compile it with:

```
cd ~/example/hello-world-program
gcc -o hello-world hello.c
```

There’s no technical reason for picking C for this example – the language doesn’t matter. It’s the binary we will be distributing in our deb package.

## Step 1: Creating a  `deb`  Package

Debian, and Debian-based Linux distributions use  `.deb`  packages to package and distribute programs. To start we will create a directory in the form:

```
<package-name>_<version>-<release-number>_<architecture>
```

where the

-   `package-name`  is the name of our package,  `hello-world`  in our case,
-   `version`  is the version of the software,  `0.0.1`  in our case,
-   `release-number`  is used to track different releases of the same software version; it’s usually set to  `1`, but hypothetically if there was an error in the packaging (e.g. a file was missed, or the description had an error in it, or a post-install script was wrong), this number would be increased to track the change, and
-   `architecture`  is the target architecture of the platform,  `amd64`  in this example; however if your package is architecture-independent (e.g. a python script), then you can set this to  `all`.

Theoretically, the directory doesn’t have to follow this naming convention; however some of the tools we will use later in the tutorial require this directory naming convention.

So for this example, we’ll create the directory with:

```
mkdir -p ~/example/hello-world_0.0.1-1_amd64
```

This directory will be the root of the package. Since we want our  `hello-world`  binary to be installed system wide, we’ll have to store it under  `usr/bin/hello-world`  with the following commands:

```
cd ~/example/hello-world_0.0.1-1_amd64
mkdir -p usr/bin
cp ~/example/hello-world-program/hello-world usr/bin/.
```

Each package requires a  `control`  file which needs to be located under the  `DEBIAN`  directory. You can copy and paste the following to create one:

```
mkdir -p ~/example/hello-world_0.0.1-1_amd64/DEBIAN

echo "Package: hello-world
Version: 0.0.1
Maintainer: example <example@example.com>
Depends: libc6
Architecture: amd64
Homepage: http://example.com
Description: A program that prints hello" \
> ~/example/hello-world_0.0.1-1_amd64/DEBIAN/control
```

Note that we’re assuming an amd64 Architecture for this tutorial, if your binary is for a different architecture, adjust accordingly. If you’re distributing a platform-independent package, you can set the architecture to  `all`.

By this point you should have created two files:

```
~/example/hello-world_0.0.1-1_amd64/usr/bin/hello-world
~/example/hello-world_0.0.1-1_amd64/DEBIAN/control
```

To build the .deb package, run:

```
dpkg --build ~/example/hello-world_0.0.1-1_amd64
```

This will output a deb package under  `~/example/hello-world_0.0.1.deb`.

You can inspect the info of the deb by running:

```
dpkg-deb --info ~/example/hello-world_0.0.1.deb
```

which will show:

```
new Debian package, version 2.0.
size 2832 bytes: control archive=336 bytes.
    182 bytes,     7 lines      control
Package: hello-world
Version: 0.0.1
Maintainer: example <example@example.com>
Depends: libc6
Architecture: amd64
Homepage: http://example.com
Description: A program that prints hello
```

You can also view the contents by running:

```
dpkg-deb --contents ~/example/hello-world_0.0.1.deb
```

which will show:

```
drwxrwxr-x alex/alex         0 2021-05-17 16:21 ./
drwxrwxr-x alex/alex         0 2021-05-17 16:18 ./usr/
drwxrwxr-x alex/alex         0 2021-05-17 16:18 ./usr/bin/
-rwxrwxr-x alex/alex     16696 2021-05-17 16:18 ./usr/bin/hello-world
```

It’s also possible to run  `less ~/example/hello-world_0.0.1.deb`  which will output the above information, thanks to  `/bin/lesspipe`.

This package can then be installed using the  `-f`  option under  `apt-get install`:

```
sudo apt-get install -f ~/example/hello-world_0.0.1-1_amd64.deb
```

Then once installed, you can verify it works with commands like:

```
which hello-world
```

and

```
hello-world
```

which should output  `/usr/bin/hello-world`  and  `hello packaged world`  respectively.

Finally, if you want to remove it, you can run:

```
sudo apt-get remove hello-world
```

This concludes the first step of building a  `.deb`  package. If you have access to an existing apt repository, you could submit the deb to the repository maintainer, and call it a day.

