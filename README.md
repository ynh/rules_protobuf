# `rules_protobuf` [![Build Status](https://travis-ci.org/pubref/rules_protobuf.svg?branch=master)](https://travis-ci.org/pubref/rules_protobuf)

Bazel skylark rules for building [protocol buffers][protobuf-home]
with +/- gRPC support on (osx, linux) :sparkles:.

[bazel_image]: https://github.com/pubref/rules_protobuf/blob/master/images/bazel.png
[wtfcat_image]: https://github.com/pubref/rules_protobuf/blob/master/images/wtfcat.png
[grpc_image]: https://github.com/pubref/rules_protobuf/blob/master/images/gRPC.png

<table border="0"><tr>
<td><img src="https://github.com/pubref/rules_protobuf/blob/master/images/bazel.png" width="190"/></td>
<td><img src="https://github.com/pubref/rules_protobuf/blob/master/images/wtfcat.png" width="190"/></td>
<td><img src="https://github.com/pubref/rules_protobuf/blob/master/images/gRPC.png" width="190"/></td>
</tr><tr>
<td>Bazel</td>
<td>rules_protobuf</td>
<td>gRPC</td>
</tr></table>

### How is this related to the proto\_library rules within bazel itself?

These rules sprung out of a need to have protobuf support when there
was limited exposed and documented proto generation capabilities in
the main bazel repository.  This is a moving target.  The main goals
of this project are to:

1. Provide `protoc`, the protocol buffer compiler
   ([v3.1.x<sup>52ab</sup>](https://github.com/google/protobuf/commit/52ab3b07ac9a6889ed0ac9bf21afd8dab8ef0014)).

2. Provide the language-specific plugins.

3. Provide the necessary libraries and dependencies for gRPC support,
   when possible.

4. Provide an extensible `proto_language` abstraction (used in
   conjunction with the `proto_compile` rule) to generate outputs for
   current and future custom protoc plugins not explicitly provided
   here.

### Rules

| Language                     | Compile <sup>1</sup>  | Build <sup>2</sup> | gRPC <sup>3</sup> |
| ---------------------------: | -----------: | --------: | -------- |
| [C++](cpp)                   | [cc_proto_compile](cpp#cc_proto_compile) | [cc_proto_library](cpp#cc_proto_library) | [v1.0.1](https://github.com/grpc/grpc/releases/tag/v1.0.1) |
| [C#](csharp)                 | [csharp_proto_compile](csharp#csharp_proto_compile) | [csharp_proto_library](csharp#csharp_proto_library) | [1.0.0](https://www.nuget.org/packages/Grpc/) |
| [Closure](closure)           | [closure_proto_compile](js#closure_proto_compile) | [closure_proto_library](js#closure_proto_library)          |  |
| [Go](go)                     | [go_proto_compile](go#go_proto_compile) | [go_proto_library](go#go_proto_library) | [v1.0.5](https://github.com/grpc/grpc-go/releases/tag/v1.0.5) |
| [Go (gogo)](gogo)            | [gogo_proto_compile](gogo#gogo_proto_compile) | [gogo_proto_library](gogo#gogo_proto_library) | [8d70fb](https://github.com/grpc/grpc-go/commit/8d70fb3182befc465c4a1eac8ad4d38ff49778e2) |
| [gRPC gateway](grpc_gateway) | [grpc_gateway_proto_compile](grpc_gateway#grpc_gateway_proto_compile)<br/>[grpc_gateway_swagger_compile](grpc_gateway#grpc_gateway_swagger_compile)   | [grpc_gateway_proto_library](grpc_gateway#grpc_gateway_proto_library)<br/>[grpc_gateway_binary](grpc_gateway#grpc_gateway_binary) | [v1.0.4](https://github.com/grpc/grpc-go/releases/tag/v1.0.4) |
| [Java](java)                 | [java_proto_compile](java#java_proto_compile) | [java_proto_library](java#java_proto_library) | [v1.0.1](https://github.com/grpc/grpc-java/releases/tag/v1.0.1) |
| [Node](node)                 | [node_proto_compile](js#node_proto_compile) | [node_proto_library](js#node_proto_library)          | [1.0.0](https://www.npmjs.com/package/grpc) |
| [Objective-C](objc) | [objc_proto_compile](objc#objc_proto_compile) | [objc_proto_library](objc#objc_proto_library) <sup>4</sup> | [v1.0.0<sup>673f</sup>](https://github.com/grpc/grpc/commit/673fa6c88b8abd542ae50c4480de92880a1e4777) |
| [Python](python)             | [py_proto_compile](python#py_proto_compile)         |           | [v1.0.0<sup>673f</sup>](https://github.com/grpc/grpc/commit/673fa6c88b8abd542ae50c4480de92880a1e4777) |
| [Ruby](ruby)                 | [ruby_proto_compile](ruby#ruby_proto_compile)          |           | [v1.0.0<sup>673f</sup>](https://github.com/grpc/grpc/commit/673fa6c88b8abd542ae50c4480de92880a1e4777) |
| Custom [proto_language](protobuf#proto_language) | [proto_compile](protobuf#proto_compile) | |  |

1. Support for generation of protoc outputs via `proto_compile()`
   rule.

2. Support for generation + compilation of outputs with protobuf
   dependencies.

3. gRPC support.

4. Highly experimental (probably not functional yet). A
   work-in-progress for those interested in contributing further work.


# Usage

## 1. Install Bazel

These are build rules for [bazel][bazel-home].  If you have not already
installed `bazel` on your workstation, follow the
[bazel instructions][bazel-install].

**Bazel 0.3.1 or above is required for go support.**

> Note about protoc and related tools: bazel and rules_protobuf will
> download or build-from-source all required dependencies, including
> the `protoc` tool and required plugins.  If you do already have
> these tools installed on your workstation, bazel will *not* use them.

## 2. Add rules_protobuf your WORKSPACE

Specify the language(s) you'd like use by loading the
language-specific `*_proto_repositories` rule(s):

```python
git_repository(
  name = "org_pubref_rules_protobuf",
  remote = "https://github.com/pubref/rules_protobuf",
  tag = "v0.7.1",
  #commit = "..." # alternatively, latest commit on master
)

load("@org_pubref_rules_protobuf//java:rules.bzl", "java_proto_repositories")
java_proto_repositories()

load("@org_pubref_rules_protobuf//cpp:rules.bzl", "cpp_proto_repositories")
cpp_proto_repositories()

load("@org_pubref_rules_protobuf//java:rules.bzl", "go_proto_repositories")
go_proto_repositories()
```

Several languages have other `rules_*` dependencies that you'll need
to load before the `*_proto_repositories()` function is invoked:

| Language | Requires |
| ---:     | :---     |
| go_proto_repositories | [rules_go](https://github.com/bazelbuild/rules_go) |
| gogo_proto_repositories | [rules_go](https://github.com/bazelbuild/rules_go) |
| grpc_gateway_proto_repositories | [rules_go](https://github.com/bazelbuild/rules_go) |
| closure_proto_repositories | [rules_closure](https://github.com/bazelbuild/rules_closure) |
| csharp_proto_repositories | [rules_dotnet](https://github.com/bazelbuild/rules_dotnet) |
| node_proto_repositories | [rules_node](https://github.com/pubref/rules_node) |

If you're only interested in the `proto_compile` rule and not any
language-specific rules, just load the generic `proto_repositories`
rule.  This provides the minimal set of dependencies (only the
`protoc` tool).

```python
load("@org_pubref_rules_protobuf//protobuf:rules.bzl", "proto_repositories")
proto_repositories()
```

## 3. Add \*\_proto\_\* rules to your BUILD files

To build a java-based gRPC library:

```python
load("@org_pubref_rules_protobuf//java:rules.bzl", "java_proto_library")

java_proto_library(
  name = "protolib",
  protos = [
    "my.proto"
  ],
  with_grpc = True,
  verbose = 1, # 0=no output, 1=show protoc command, 2+ more...
)
```

## Examples

To run the examples & tests in this repository, clone it to your
workstation.

```sh
# Clone this repo
$ git clone https://github.com/pubref/rules_protobuf

# Go to examples/helloworld directory
$ cd rules_protobuf/examples/helloworld

# Run all tests
$ bazel test examples/...

# Build a server
$ bazel build cpp/server

# Run a server from the command-line
$ $(bazel info bazel-bin)/examples/helloworld/cpp/server

# Run a client
$ bazel run go/client
$ bazel run cpp/client
$ bazel run java/org/pubref/rules_closure/examples/helloworld/client:netty
```

# Overriding or excluding WORKSPACE dependencies

To load alternate versions of dependencies, pass in a
[dict][skylark-dict] having the same overall structure of a
[deps.bzl][protobuf/deps.bzl] file.  Entries having a matching key will
override those found in the file.  For example, to load a different
version of https://github.com/golang/protobuf, provide a different
commit ID:

```python
load("@org_pubref_rules_protobuf//go:rules.bzl", "go_proto_repositories")
go_proto_repositories(
  overrides = {
    "com_github_golang_protobuf": {
      "commit": "2c1988e8c18d14b142c0b472624f71647cf39adb", # Aug 8, 2016
    }
  },
)
```

You may already have some external dependencies already present in
your workspace that rules_protobuf will attempt to load, causing a
collision.  To prevent rules_protobuf from loading specific external
workspaces, name them in the `excludes` list:

```python
go_proto_repositories(
  excludes = [
    "com_github_golang_glog",
  ]
)
```

To completely replace the set of dependencies that will attempt to be
loaded, you can pass in a full `dict` object to the `lang_deps`
attribute.

```python
go_proto_repositories(
  lang_deps = {
    "com_github_golang_glog": {
      ...
    },
  },
)
```

There are several language --> language dependencies as well. For
example, `python_proto_repositories` and `ruby_proto_repositories`
(and more) internally call the `cpp_proto_repositories` rule to
provide the grpc plugins.  To suppress this (and have better control
in your workspace), you can use the `omit_cpp_repositories=True`
option.

# Proto B --> Proto A dependencies

Use the `proto_deps` attribute to name proto rule dependencies. Use of
`proto_deps` implies you're using imports, so read on...

## Imports

In all cases, these rules will include a `--proto_path=.` (`-I.`)
argument.  This is functionally equivalent to `--proto_path=$(bazel
info execution_root)`.  Therefore, when the protoc tool is invoked, it
will 'see' whatever directory structure exists at the bazel execution
root for your workspace.  To better learn what this looks like, `cd
$(bazel info execution_root)` and look around.  In general, it
contains all your sourcefiles as they appear in your workspace with an
additional `external/WORKSPACE_NAME` directory for all dependencies
used.

This has implications for import statements in your protobuf
sourcefiles, if you use them.  The two cases to consider are imports
*within* your workspace (referred to here as *'internal' imports*),
and imports of other protobuf files in an external workspace
(*external imports*).

### Internal Imports

Internal imports should require no additional parameters if your
import statements follow the same directory structure of your
workspace.  For example, the
`examples/helloworld/proto/helloworld.proto` file imports the
`examples/proto/common.proto` file.  Since this matches the workspace
directory structure, `protoc` can find it, and no additional arguments
to a `cc_proto_library` are required for protoc code generation step.

Obviously, importing a file does not mean that code will be generated
for it.  Therefore, *use of the imports attribute implies that the
generated files for the imported message or service already exist
somewhere that can be used as a dependency some other library rule*
(such as `srcs` for `java_library`).

Rather than using `imports`, it often make more sense to declare a
dependency on another proto_library rule via the `proto_deps`
attribute.  This makes the import available to the calling rule and
performs the code generation step.  For example, the
`cc_proto_library` rule in `examples/helloworld/proto:cpp` names the
`//examples/proto:cpp`'s `cc_proto_library` rule in its `proto_deps`
attribute to accomplish both code generation and compilation of object
files for the proto chain.

### External Imports

The same logic applies to external imports.  The two questions to ask
yourself when setting up your rules are:

[Question 1]: *Can protoc "see" the imported file?* In order to satisfy this
requirement, pass in the full path of the required file(s) relative to
the execution root where protoc will be run.  For example, the the
well-known `descriptor.proto` could be made visible to protoc via:

```python
java_proto_library(
  name = 'fooprotos',
  protos = 'foo.proto`,
  imports = [
      "external/com_github_google_protobuf/src/",
    ],
    inputs = [
      "@com_github_google_protobuf//:well_known_protos",
    ],
)
```

This would be imported as `import "google/protobuf/descriptor.proto"`
given that the file
`@com_github_google_protobuf/src/google/protobuf/descriptor.proto` is
in the package `google.protobuf`.

[Question 2]: *Can the `cc_proto_library` rule "see" the generated protobuf files*?
(in this case `descriptor.pb.{h,cc}`.  Just because the file was
imported does not imply that protoc will generate outputs for it, so
somewhere in the `cc_library` rule dependency chain these files must
be present.  This could be via another `cc_proto_library` rule defined
elswhere, or a some other filegroup or label list.  If the source is
another `cc_proto_library` rule, specify that in the `proto_deps`
attribute to the calling `cc_proto_library` rule.  Otherwise, pass a
label that includes the (pregenerated) protobuf files to the `deps`
attribute, just as you would any typical `cc_library` rule.

**Important note about sandboxing**: simply stating the path where
  protoc should look for imports (via the `imports` attribute) is not
  enough to work with the bazel sandbox.  Bazel is very particular
  about needing to know *exactly* which inputs are required for a
  rule, and *exactly* what output files it generates.  If an input is
  not declared, it will not be exposed in the sandbox.  Therefore, we
  have to provide both the import path *and* a label-generating rule
  in the `inputs` attribute that names the files we want available in
  the sandbox (given here by `:well_known_protos`).

If you are having problems, put `verbose={1,2,3}` in your build rule
and/or disable sandboxing with `--spawn_strategy=standalone`.

> Note: bazel will likely a breaking change to the way external
> repositories are laid out in the execution root in future versions
> of bazel.  This will likely affect the naming of import paths when
> this change occurs.

# Contributing

Contributions welcome; please create Issues or GitHub pull requests.

# Credits

* [@yugui][yugui]: Primary source for the go support from [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway/blob/e958c5db30f7b99e1870db42dd5624322f112d0c/examples/bzl/BUILD).

* [@mzhaom][mzhaom]: Primary source for the skylark rule (from
  <https://github.com/mzhaom/trunk/blob/master/third_party/grpc/grpc_proto.bzl>).

* [@jart][jart]: Overall repository structure and bazel code layout
  (based on [rules_closure]).

* [@korfuri][korfuri]: Prior research on travis-ci integration.

* Much thanks to all the members of the bazel, protobuf, and gRPC teams.

[yugui]: http://github.com/yugui "Yuki Yugui Sonoda"
[jart]: http://github.com/jart "Justine Tunney"
[mzhaom]: http://github.com/mzhaom "Ming Zhao"
[korfuri]: http://github.com/korfuri "Uriel Korfa"

[protobuf-home]: https://developers.google.com/protocol-buffers/ "Protocol Buffers Developer Documentation"
[bazel-home]: http://bazel.io "Bazel Homepage"
[bazel-install]: http://bazel.io/docs/install.html "Bazel Installation"
[rules_closure]: http://github.com/bazelbuild/rules_closure "Rules Closure"
[rules_go]: http://github.com/bazelbuild/rules_go "Rules Go"
[grpc-gateway-home]:https://github.com/grpc-ecosystem/grpc-gateway

[bazel_image]: https://github.com/pubref/rules_protobuf/blob/master/images/bazel.png
[wtfcat_image]: https://github.com/pubref/rules_protobuf/blob/master/images/wtfcat.png
[grpc_image]: https://github.com/pubref/rules_protobuf/blob/master/images/gRPC.png

[repositories.bzl]: protobuf/internal/repositories.bzl

[skylark-dict]: https://www.bazel.io/docs/skylark/lib/dict.html "Skylark Documentation for dict"
[skylark-string]: https://www.bazel.io/docs/skylark/lib/attr.html#string "Skylark string attribute"
[skylark-string_list]: https://www.bazel.io/docs/skylark/lib/attr.html#string_list "Skylark string_list attribute"
[skylark-string_list_dict]: https://www.bazel.io/docs/skylark/lib/attr.html#string_list_dict "Skylark string_list_dict attribute"
