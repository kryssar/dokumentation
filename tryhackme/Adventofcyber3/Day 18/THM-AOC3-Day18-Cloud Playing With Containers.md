---
title: Advent Of Cyber 3 - Day 18
date: "2021-12-18"
description: Docker, Containers
tldr: Docker
tags: [docker,ecr,tryhackme,adventofcyber3,cloud]

---

AWS Elastic Container Registry - ECR Public Gallery , playing around with docker images.

The grinch enterprises have a public ECR that looks suspicious: [Grinch Enterprises](https://gallery.ecr.aws/h0w1j9u3/grinch-aoc)

using windows docker or linux docker, pull the image with: 
```shell 
docker pull public.ecr.aws/h0w1j9u3/grinch-aoc:latest

docker pull public.ecr.aws/h0w1j9u3/grinch-aoc:latest  
latest: Pulling from h0w1j9u3/grinch-aoc  
7b1a6ab2e44d: Pull complete  
7181c3c4941b: Pull complete  
148b30b9ae2d: Pull complete  
6f5a7c388565: Pull complete  
ef099323cb4a: Pull complete  
de5bf7e2abf0: Pull complete  
455d5424d859: Pull complete  
b1ee65a7e02a: Pull complete  
a47021107475: Pull complete  
Digest: sha256:593c79eaaa1a905c533e389b0034022e074969da3936df648172c4efc8d421d8  
Status: Downloaded newer image for public.ecr.aws/h0w1j9u3/grinch-aoc:latest  
public.ecr.aws/h0w1j9u3/grinch-aoc:latest
```

and then run it with: `docker run -it public.ecr.aws/h0w1j9u3/grinch-aoc:latest`

with this command we'll get a shell directly into the docker-container and can interact with it directly as if we were on the server itself.

there's nothing in the home-folder of the container-user. But there's some interesting environment variables: 
```shell
$ printenv  
HOSTNAME=949c288f8dc1  
HOME=/home/newuser  
OLDPWD=/home  
TERM=xterm  
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin  
api_key=a90eac086fd049ab9a08374f65d1e977  
PWD=/
```

## Further analysis of the docker container

exit from the shell and save the running container as a tar-file. 

```shell
exit
mkdir aoc && cd $_
docker save -o aoc.tar public.ecr.aws/h0w1j9u3/grinch-aoc:latest
```

unpack the saved container to see what files are contained in it. 
```shell
tar xf aoc.tar
```

in case `jq` isn't installed, get it. 
```shell
sudo apt install -y jq
```

the `manifest.json` file includes information that we want to investigate further.

```shell 
cat manifest.json | jq
```

the `jq` command formats it in JSON so it's easier to read:

```json
[
  {
    "Config": "f886f00520700e2ddd74a14856fcc07a360c819b4cea8cee8be83d4de01e9787.json",
    "RepoTags": [
      "public.ecr.aws/h0w1j9u3/grinch-aoc:latest"
    ],
    "Layers": [
      "06ec107a7c3909292f0730a926f0bf38071c4b930618cb2480e53584f4b60777/layer.tar",
      "df316f55e15855625078e9ae6f6812c2e83164feabacf457a1c0b4d332622806/layer.tar",
      "6a750165b0eb6d29d0b4e4cd054096a0a295fc606d10fced8ab7389adb7dd13f/layer.tar",
      "72ed4c44c5d38246a6ff7938a3e48c0c68f8543bb30be8a773f02b5d055362ce/layer.tar",
      "9eafea9736f44679aac855b58c0ad10e476c41a3b07eb99718e19ee79f512b4f/layer.tar",
      "b11674d3410c42d488ff618e486fcd7263ad6029798de0d7526871e7945969d2/layer.tar",
      "349a6efa7f944faf10f4d35fa2c11089e36cc474b3d91a3ce39df6e84e9c0452/layer.tar",
      "249855506821100cff82e4b2ce1f920b51bcff2b3272de2b9636eb4c83572beb/layer.tar",
      "ac6f2352e5431dcf74287b5c88340c9f3ae1b7b2c1bfb08fe063602bb49ab591/layer.tar"
    ]
  }
]
```

go through the various layers that were unpacked into folders with `layer.tar` files inside of them. In the folder `249855506821100cff82e4b2ce1f920b51bcff2b3272de2b9636eb4c83572beb`

there's a `root/envconsul` folder that looks interesting. Extract that folder with 7-zip and dig further in the config files for `envconsul` named `config.hcl` 

looking for a token, we can see that they forgot to clean it out from this particular layer: 
```shell
┌──(kryssar㉿kali)-[/mnt/…/day18/aoc/249855506821100cff82e4b2ce1f920b51bcff2b3272de2b9636eb4c83572beb/envconsul]
└─$ grep token config.hcl                                                                              
  # This is the token to use when communicating with the Vault server.
  # assumption that you provide it with a Vault token; it does not have the
  # incorporated logic to generate tokens via Vault's auth methods.
  token = "7095b3e9300542edadbc2dd558ac11fa"
```

command to list container images stored in local container registry: `docker images`

command to save docker image as tar archive: `docker save`

name of the file for configuration, repo tags and layer hash values: `manifest.json`

token value: `7095b3e9300542edadbc2dd558ac11fa`


EOF



