# docker-compose push 命令的含义

## 一|疑问

`docker-compose push --help` 查看说明时，并不明确。

官方文档 [docker-compose push](https://docs.docker.com/compose/reference/push/) 说的明明白白 ，是把 `docker-compose.yml` 文件中的 `Service` push 到 Registry 中去。

但是查了一圈，发现 `docker-compose push` 命令并不是这样。而是和 `docker stack` 相关，在 V3 格式中会被 `docker stack deploy` 替代。

## 二|释疑

仔细端详以下文档后：

- 官方文档 [docker-compose push](https://docs.docker.com/compose/reference/push/)
- [build](https://docs.docker.com/compose/compose-file/#build)
- [image](https://docs.docker.com/compose/compose-file/#image)
- [how-do-i-define-the-name-of-image-built-with-docker-compose](https://stackoverflow.com/questions/32230577/how-do-i-define-the-name-of-image-built-with-docker-compose)
- [container_name](https://docs.docker.com/compose/compose-file/#container_name)

发现关键：

    # build
    If you specify image as well as build, then Compose names the built image with the webapp and optional tag specified in image:

    # image
    If the image does not exist, Compose attempts to pull it, unless you have also specified build, in which case it builds it using the specified options and tags it with the specified tag.

docke-compose.yml 的 Service 定义，同时存在 `build` 和 `image` 时（V2+），`image` 用于指定构建出来的镜象的 tag。

这样一来，[docker-compose push](https://docs.docker.com/compose/reference/push/) 文档中的示例，就合情合理了：

```yml
version: '3'
services:
  service1:
    build: .
    image: localhost:5000/yourimage  # goes to local registry

  service2:
    build: .
    image: youruser/yourimage  # goes to youruser DockerHub registry
```

`docker-compose build` 和 `docker-compose push` 命令执行完之后，确实能把构建出来的镜象 push 到 Registry 中去。

## 三|参考

- [docker-compose push](https://docs.docker.com/compose/reference/push/)
- [What is `docker-compose push` supposed to do?](https://github.com/docker/compose/issues/4283)
- [Don't push local images](https://github.com/docker/compose/issues/6112)
- [build](https://docs.docker.com/compose/compose-file/#build)
- [image](https://docs.docker.com/compose/compose-file/#image)
- [how-do-i-define-the-name-of-image-built-with-docker-compose](https://stackoverflow.com/questions/32230577/how-do-i-define-the-name-of-image-built-with-docker-compose)
- [container_name](https://docs.docker.com/compose/compose-file/#container_name)