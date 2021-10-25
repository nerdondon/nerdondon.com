# nerdondon.com

New blog based on [Hugo](https://gohugo.io/).

## Build and Run

With the @klakegg docker image:

```shell
docker run --rm -it -v ${PWD}:/src klakegg/hugo:0.88.0-ext-debian
```

Server out of port 1313:

```shell
docker run --rm -it -v ${PWD}:/src -p 1313:1313 klakegg/hugo:0.88.0-ext-debian server
```

### Use the image as a shell

```shell
docker run --rm -it -v ${PWD}:/src -p 1313:1313 klakegg/hugo:0.88.0-ext-debian shell
```

## Dependencies

Update submodules (e.g. for themes) with the following command:

```shell
git submodule update --remote
```
