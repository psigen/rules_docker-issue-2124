load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

rules_docker_sha = "d6bdd1790f839cbb111be4e66315a0cc7f280637111e32bc2e103e2602b4c6ad"

rules_docker_tag = "b73960f1f98a2d3a515d79ce97ad7f920f562b6d"

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
