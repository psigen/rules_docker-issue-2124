local_repository(
    name = "io_bazel_rules_docker",
    path = "../rules_docker",
)

# http_archive(
#     name = "io_bazel_rules_docker",
#     sha256 = "b1e80761a8a8243d03ebca8845e9cc1ba6c82ce7c5179ce2b295cd36f7e394bf",
#     urls = ["https://github.com/bazelbuild/rules_docker/releases/download/v0.25.0/rules_docker-v0.25.0.tar.gz"],
# )

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
