# Rust in Docker


## Install rust, compile, run
This is a single command that uses a here-Doc within a here-doc pattern.
The outer here-doc sends commands to the container to install the rust compiler.
The inner here-doc sends code to the rust compiler that is inside the container.
```bash
<< 'eof1' docker run --rm -i -w /tmp/ ubuntu
  export DEBIAN_FRONTEND=noninteractive
  apt-get update
  apt-get install -y rustc
  cat <<'eof' | rustc -o hello -
fn main() {
    println!("Hello, World");
}
eof
  ./hello
eof1
```


## Install, create image, compile, run
This creates a running instance then creates an image from that instance.
Finally, the new image is used to run a separate container, which uses the here-doc within a here-doc pattern.
```bash
<< 'eof' docker run --rm -i --name rustc ubuntu &
  export DEBIAN_FRONTEND=noninteractive
  apt-get update
  apt-get install -y rustc
  echo === ready to commit to image
  sleep inf
eof

docker container commit rustc rust:latest

docker container stop rustc

<< 'eof1' docker run --rm -i rust
  cat <<'eof' | rustc -o hello -
fn main() {
    println!("Hello, World");
}
eof
  ./hello
eof1
```

## Build image, compile, run
This is an example of using an "in-line" Dockerfile to build an image and then using the resulting image to run a container.

```bash
<< 'eof' docker build --tag rust-dev -
  FROM ubuntu
  ENV  DEBIAN_FRONTEND=noninteractive
  RUN  apt-get update
  RUN  apt-get install -y rustc vim tree rsync zip less
  RUN  echo == done
eof

<< 'eof1' docker run --rm -i rust-dev
  <<'eof' rustc -o hello - ; ./hello
fn main() {
    println!("Hello, World");
}
eof
eof1
```

## Interactive rust instance
This uses the image from the previous build to run an interactive container.
Notice that I don't run the container as a daemon.  Just my personal preference.
I can stop and remove the container by simply stopping it.
I also prefer using exec to "enter" the container.
Again, my person preference over detaching ( ^P^Q ) and attaching. 

```bash
docker run --rm --name rust -w /tmp rust-dev sleep inf & sleep 1

<<'eof' docker exec -i rust /bin/bash -c 'cat > ~/.bash_aliases'
  alias cls='clear';
  alias dir='ls -la';
  alias h='history';
  alias more='less -iX';
  export HISTCONTROL=ignoredups:ignorespace;
  export HISTFILESIZE=50000;
  export HISTSIZE=50000;
  export HISTTIMEFORMAT='%t%F %T%t';
  export PAGER='less -iX ';
  export IGNOREEOF=20;
  export PS1='\u@\h: \w\n\$ '
eof

docker exec -it rust /bin/bash

docker stop rust
```


