<!--
theme: uncover
footer: Thomas Eizinger: @wheezle@hachyderm.io
paginate: true
-->

<style scoped>
* {
  color: white;
}
</style>


# Scaling library workspaces

![bg](https://i.postimg.cc/9Qx71kPT/DALL-E-2023-04-12-15-54-26-a-photo-of-a-scale-of-justice-that-is-slightly-imbalanced-the-scale-sh.png)

<!--

Experience talk,
Share learnings from maintaining a big library workspace,
Spoiler: As with scaling anything, you hit interesting limits

-->

---
<!--
footer: ""
-->

# Context

- Library vs. binary workspaces
- crates represent hard interfaces

```
- github.com/libp2p/rust-libp2p
|
|-- swarm/
|   |
|   -- Cargo.toml
|-- noise/
|   |
|   -- Cargo.toml
|
Cargo.toml # <-- workspace manifest
```

<!--

To start with, clarify what we are talking about.

A cargo workspace is a _virtual_ manifest that is not a crate itself.
Allows to compile multiple crates at once.
"monorepo"
In a library workspace, we ship all crates to crates.io
In a binary workspace, we just ship the binary of one of the crates

-->

---

![bg contain](https://i.postimg.cc/5t4yLtdm/graph.png)

---

# Why so many crates?

* Modular network stack
* Swap out transports / multiplexer / encryption protocol
* Built-in modules for DHT, pubsub, ...

---

# Continuous integration

* cargo doesn't parallelise execution across crates 👎️
* Features are unified across a build-graph 👎️
* Dynamically create one CI job per crate

* ```bash
  cargo metadata --format-version=1 --no-deps | \
   jq -c '.packages | map(select(.publish == null) | .name)'
  ```
  ```yml
  matrix:
    crate: ${{ fromJSON(...) }}
  ```

---

# Releases and bumping versions

* Which crates need to be released?
* What is the next version?
* Tooling:
  * `cargo semver-checks`
  * `cargo release`
  * Maybe soon: `cargo semverlog`

<!--

Another tricky topic is the versioning of crates.
You'd think deciding the next version is easy.

Bump as soon as the change is made.
Merge conflicts in CHANGELOG files.

-->

---

# Breaking changes

![bg contain](https://i.postimg.cc/rm2QvgNZ/Lu-KYdrz-Imgur.png)

---

# Syncing versions across workspace crates

```toml
# file: swarm/Cargo.toml
[package]
name = "libp2p-swarm"
version = "0.42.2"
```

```toml
# file: kad/Cargo.toml
[dependencies]
libp2p-swarm = { version = "0.42.0", path = "../swarm" }
```

<!--
Say you bump the patch version correctly for a non-breaking change.
Are you depending on the correct version everywhere?
-->

---

![bg contain](https://i.postimg.cc/5t4yLtdm/graph.png)

---

# Workspace inheritance (Rust 1.64)

```toml
# file: Cargo.toml
[workspace.dependencies]
libp2p-swarm = { version = "0.42.0", path = "../swarm" }
```

```toml
# file: kad/Cargo.toml
[dependencies]
libp2p-swarm = { workspace = true }
```


---

# Learnings

* Keep your workspaces small
* Have many automated checks
* Have good crate boundaries

---

# Thanks for listening

- Repository: github.com/libp2p/rust-libp2p
- Mastodon: @wheezle@hachyderm.io
