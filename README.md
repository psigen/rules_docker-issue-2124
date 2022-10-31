# rules_docker-issue-2124

This is a replication of the issue described in:
https://github.com/bazelbuild/rules_docker/issues/2124

To replicate the issue, run the following command:

```bash
$ bazel run //:image -- -e=baz=C -- --beep
```

This produces the following incorrect output:

```bash
$ bazel run //:image -- -e=baz=C -- --beep
DEBUG: /home/berkshiregrey.com/pras/.cache/bazel/_bazel_pras/dcecc182d8325095d3b3ff25e867cee5/external/io_bazel_rules_docker/container/image.bzl:353:10: impl orig entrypoint: ["/usr/bin/java", "-cp", "/app/__main__/liblib.jar:/app/__main__/image.binary.jar:/app/__main__/image.binary", "demo.Main", "-e=foo=A", "-e=bar=B"]
DEBUG: /home/berkshiregrey.com/pras/.cache/bazel/_bazel_pras/dcecc182d8325095d3b3ff25e867cee5/external/io_bazel_rules_docker/container/image.bzl:354:10: impl orig cmd: None
DEBUG: /home/berkshiregrey.com/pras/.cache/bazel/_bazel_pras/dcecc182d8325095d3b3ff25e867cee5/external/io_bazel_rules_docker/container/image.bzl:377:10: impl used entrypoint: ["/usr/bin/java", "-cp", "/app/__main__/liblib.jar:/app/__main__/image.binary.jar:/app/__main__/image.binary", "demo.Main", "-e=foo=A", "-e=bar=B"]
DEBUG: /home/berkshiregrey.com/pras/.cache/bazel/_bazel_pras/dcecc182d8325095d3b3ff25e867cee5/external/io_bazel_rules_docker/container/image.bzl:378:10: impl used cmd: []
DEBUG: /home/berkshiregrey.com/pras/.cache/bazel/_bazel_pras/dcecc182d8325095d3b3ff25e867cee5/external/io_bazel_rules_docker/container/image.bzl:155:10: _add_create_image_config_args args: -outputConfig bazel-out/k8-fastbuild-ST-4a519fd6d3e4/bin/image.0.config -outputManifest bazel-out/k8-fastbuild-ST-4a519fd6d3e4/bin/image.0.manifest -entrypoint /usr/bin/java -entrypoint -cp -entrypoint /app/__main__/liblib.jar:/app/__main__/image.binary.jar:/app/__main__/image.binary -entrypoint demo.Main -entrypoint -e=foo=A -entrypoint -e=bar=B -env JAVA_RUNFILES=/app -layerDigestFile @bazel-out/k8-fastbuild-ST-4a519fd6d3e4/bin/image-layer.tar.sha256 -baseConfig external/java/image/config.json -architecture amd64 -operatingSystem linux
DEBUG: /home/berkshiregrey.com/pras/.cache/bazel/_bazel_pras/dcecc182d8325095d3b3ff25e867cee5/external/io_bazel_rules_docker/container/image.bzl:247:10: _assemble_image_digest args:
DEBUG: /home/berkshiregrey.com/pras/.cache/bazel/_bazel_pras/dcecc182d8325095d3b3ff25e867cee5/external/io_bazel_rules_docker/container/image.bzl:545:10: impl executable path bazel-out/k8-fastbuild-ST-4a519fd6d3e4/bin/image.executable
INFO: Analyzed target //:image (113 packages loaded, 7870 targets configured).
INFO: Found 1 target...
Target //:image up-to-date:
  bazel-out/k8-fastbuild-ST-4a519fd6d3e4/bin/image-layer.tar
INFO: Elapsed time: 1.232s, Critical Path: 0.01s
INFO: 1 process: 1 internal.
INFO: Build completed successfully, 1 total action
INFO: Build completed successfully, 1 total action
incremental_load.sh ALL: -e=foo=A -e=bar=B -e=baz=C -- --beep
Loaded image ID: sha256:c57665c0d380db630c97c9a2b20f86370653ad66ec9e0ddb60f481b3d9820e91
Tagging c57665c0d380db630c97c9a2b20f86370653ad66ec9e0ddb60f481b3d9820e91 as bazel:image
incremental_load.sh ARGS: /usr/bin/docker run -i --rm --network=host -e=foo=A -e=bar=B -e=baz=C bazel:image --beep
hello there!
[-e=foo=A, -e=bar=B, --beep]
```

## Explanation

This reproduction uses four distinct flags that should be visible in different places.

- `-e=foo=A` - this should be included in the image entrypoint and not passed by `bazel run`
- `-e=bar=B` - this should be included in the image entrypoint and not passed by `bazel run`
- `-e=baz=C` - this should be passed by `bazel run` to `docker run`
- `--beep` - this should be passed by `bazel run` to the application

We would therefore expect the following output from our debugging:

```
incremental_load.sh ARGS: /usr/bin/docker run -i --rm --network=host -e=baz=C bazel:image --beep
hello there!
[-e=foo=A, -e=bar=B, --beep]
```

Note the following problems with the actual output:

We can see that the original entrypoint includes all the flags necessary to run the command.
I.e. the entrypoint and command already include the "-e=foo=A" and "-e=bar=B" flags.

```
container/image.bzl:377:10: impl used entrypoint: ["/usr/bin/java", "-cp", "/app/__main__/liblib.jar:/app/__main__/image.binary.jar:/app/__main__/image.binary", "demo.Main", "-e=foo=A", "-e=bar=B"]
DEBUG: /home/berkshiregrey.com/pras/.cache/bazel/_bazel_pras/dcecc182d8325095d3b3ff25e867cee5/external/io_bazel_rules_docker/container/image.bzl:378:10: impl used cmd: ["/usr/bin/java", "-cp", "/app/__main__/liblib.jar:/app/__main__/image.binary.jar:/app/__main__/image.binary", "demo.Main", "-e=foo=A", "-e=bar=B"]
```

However, these same args are passed again to the incremental load script, even though they are already in the entrypoint command:

```
incremental_load.sh ARGS: /usr/bin/docker run -i --rm --network=host -e=foo=A -e=bar=B -e=baz=C bazel:image --beep
```

Finally, the _original_ args are still correctly passed to the executable _and_ appended correctly to the new argument.

```
[-e=foo=A, -e=bar=B, --beep]
```

This means that the original args added to the entrypoint worked, and the args passed to the docker-run command were unnecessary.
