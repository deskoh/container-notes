# Rootless

```sh
# Non-root
$ podman unshare cat /proc/self/uid_map
         0       1000          1
         1     100000      65536
$ podman run alpine cat /proc/self/uid_map
         0       1000          1
         1     100000      65536
```

## Reference

[Basic Setup and Use of Podman in a Rootless environment.](https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md)

[Rootless containers with Podman: The basics](https://developers.redhat.com/blog/2020/09/25/rootless-containers-with-podman-the-basics)

[Running rootless Podman as a non-root user](https://www.redhat.com/sysadmin/rootless-podman-makes-sense)

[Running rootless Podman as a non-root user](https://www.redhat.com/sysadmin/rootless-podman-makes-sense)

[Podman and user namespaces: A marriage made in heaven](https://opensource.com/article/18/12/podman-and-user-namespaces)

[https://opensource.com/article/19/2/how-does-rootless-podman-work](https://opensource.com/article/19/2/how-does-rootless-podman-work)
