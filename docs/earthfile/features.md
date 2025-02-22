# Experimental Features

Earthly makes use of feature flags to release new and experimental features.
These features must be explicitly enabled to use them.

Earthly uses [semantic versioning](http://semver.org/); once a new feature
has reached stability, a new major or minor version of Earthly will be released with
the feature enabled by default.

## Specifying Version and features

Each Earthfile should list the current earthly version it depends on using the [`VERSION`](../earthfile/earthfile.md#version) command.
The `VERSION` command was first introduced under `0.5` and is currently optional; however, it will become mandatory in a future version.

```Dockerfile
VERSION [<flags>...] <version-number>
```

### Example

To Future-proof Earthfiles it is recommended to add a `VERSION` command. Consider a case where an Earthfile is developed
against earthly `v0.5.23` and makes use of the experimental `FOR` command, the first line of the Earthfile should be:

```Dockerfile
VERSION --for-in 0.5
```

This will ensure that backwards-breaking features that are introduced in a later version will not change how this Earthfile is interpreted.

In a future release (e.g. `0.X`), the `for` command **might** be promoted from the _experimental_ stage to _stable_ stage,
at that point, version `0.X` would automatically set the `--for-in` flag to `true`, and the Earthfile could be updated
to require version `0.X` (or later), and could be rewritten as `VERSION 0.X`.


## Feature flags

| Feature flag | status | description |
| --- | --- | --- |
| `--use-copy-include-patterns` | experimental | Speeds up COPY transfers |
| `--referenced-save-only` | experimental | Changes the behavior of SAVE commands in a significant way |
| `--for-in` | experimental | Enables support for `FOR ... IN ...` commands |

All of these features flags are disabled by default in version `0.5`.

##### `--use-copy-include-patterns`

*Speeds up COPY transfers.*

When enabled, Earthly will only send the files listed for the specific [`COPY`](../earthfile/earthfile.md#copy) command.
Without this feature, Earthly sends the entire directory of files excluding files listed in the [`.earthignore` file](../earthfile/earthignore.md).

##### `--referenced-save-only`

*Changes the behavior of SAVE commands in a significant way*

When enabled, Earthly will output artifacts resulting from `SAVE ARTIFACT ... AS LOCAL ...` and images resulting from `SAVE IMAGE` and also execute `RUN --push` commands only if they are connected to the main target through a chain of `BUILD` commands.

For example, chains like these will produce outputs (and possibly push, if enabled):

* main target -> `SAVE`
* main target -> `BUILD -> SAVE`
* main target -> `BUILD -> BUILD -> SAVE`
* main target -> `BUILD -> BUILD -> BUILD -> SAVE`

While chains like these will NOT produce outputs nor would they push:

* main target -> `FROM -> SAVE`
* main target -> `COPY -> SAVE`
* main target -> `FROM -> BUILD -> SAVE`
* main target -> `BUILD -> FROM -> SAVE`
* main target -> `BUILD -> BUILD -> COPY -> SAVE`

This works the same regardless of whether the targets in the chain are remote or local.

When this feature is **disabled**, Earthly will output artifacts and images regardless of whether they are connected to the main target through a chain of `BUILD` commands, however the outputs will be subject to the following rules:

* All `SAVE ARTIFACT ... AS LOCAL ...`, with local Earthfiles will be output
* `SAVE ARTIFACT ... AS LOCAL ...` produced in remote targets will not be output
* All images with tag names (both local and remote Earthfiles) will be output
* No image will be pushed or `RUN --push` command will be executed if the target is remote

##### `--for-in`

*Enables support for `FOR ... IN ...` commands*

When enabled, Earthly will allow the use of `FOR ... IN ...` commands.
