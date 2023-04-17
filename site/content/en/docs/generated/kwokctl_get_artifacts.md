## kwokctl get artifacts

Lists binaries or images used by cluster

```
kwokctl get artifacts [flags]
```

### Options

```
      --filter string    Filter the list of (binary or image)
  -h, --help             help for artifacts
      --runtime string   Runtime of the cluster (binary or docker or kind or nerdctl)
```

### Options inherited from parent commands

```
  -c, --config stringArray   config path (default [~/.kwok/kwok.yaml])
      --name string          cluster name (default "kwok")
  -v, --v log-level          number for the log level verbosity (DEBUG, INFO, WARN, ERROR) or (-4, 0, 4, 8) (default INFO)
```

### SEE ALSO

* [kwokctl get](kwokctl_get.md)	 - Gets one of [artifacts, clusters, kubeconfig]
