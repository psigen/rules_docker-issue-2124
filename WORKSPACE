load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

rules_docker_sha = "f3a6901fe69b9a17ff94a08446e2d4950ffcd260137d9d4c93776f222bacaa22"

rules_docker_tag = "224f37b3569233c9df77cceba56280560001230d"

http_archive(
    name = "io_bazel_rules_docker",
    sha256 = rules_docker_sha,
    strip_prefix = "rules_docker-{}".format(rules_docker_tag),
    urls = ["https://github.com/psigen/rules_docker/archive/{}.zip".format(rules_docker_tag)],
)

# http_archive(
#     name = "io_bazel_rules_docker",
#     sha256 = "b1e80761a8a8243d03ebca8845e9cc1ba6c82ce7c5179ce2b295cd36f7e394bf",
#     urls = ["https://github.com/bazelbuild/rules_docker/releases/download/v0.25.0/rules_docker-v0.25.0.tar.gz"],
# )

# local_repository(
#     name = "io_bazel_rules_docker",
#     path = "../rules_docker",
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
