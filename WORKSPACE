local_repository(
    name = "io_bazel_rules_docker",
    path = "../rules_docker",
)

load("@io_bazel_rules_docker//repositories:repositories.bzl", "repositories")

repositories()

load("@io_bazel_rules_docker//repositories:deps.bzl", "deps")

deps()

load("@io_bazel_rules_docker//container:container.bzl", "container_pull")

container_pull(
    name = "java",
    registry = "gcr.io",
    repository = "distroless/java",
    tag = "latest",
)
