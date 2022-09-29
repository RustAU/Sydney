---
theme: uncover
paginate: true
---

<style>
  :root {
    --color-background: #ddd;
    --color-background-code: #ccc;
    --color-background-paginate: rgba(128, 128, 128, 0.05);
    --color-foreground: #345;
    --color-highlight: #99c;
    --color-highlight-hover: #aaf;
    --color-highlight-heading: #99c;
    --color-header: #bbb;
    --color-header-shadow: transparent;
  }

  section.lead p {
    text-align: left;
  }
</style>

# **Programming the GPU with Rust**

Cameron Duff
@SchumannSprite

---

# This runs on the GPU

```rust
use spirv_std::glam::{vec4, Vec4};

#[spirv(fragment)]
pub fn main_fs(
    output: &mut Vec4
) {
    *output = vec4(1.0, 0.0, 0.0, 1.0);
}

```

---

# Enter **Rust-GPU**

[https://github.com/EmbarkStudios/rust-gpu](https://github.com/EmbarkStudios/rust-gpu)

* Rust compiler backend
* Targets SPIR-V
* Shader code in rust using cross compilation
* The same rust code runs natively

---

### The rust compiler works as normal

* Lifetimes
* The borrow checker
* Safe/unsafe code
* Mutability
* Fearless concurrency
* Memory safety

---

### The rust toolchain works as normal

* Conditional compilation
* Macros
* Modules
* Crates
* Clippy lints
* Unit tests
* Benchmarks

---

# Standard library

[https://docs.rs/spirv-std/latest/spirv_std/](https://docs.rs/spirv-std/latest/spirv_std/)

* Primitive types from the normal std library work
* Extra types available for use in SPIR-V
* Scalars, Vectors
* Atomics
* Images, Buffers

---

# Modularity and Reusability

```rust
mod common {
    pub fn saturate(x: f32) -> f32 {
        x.max(0.0).min(1.0)
    }

    pub fn smoothstep(edge0: f32, edge1: f32, x: f32) -> f32 {
        let x = saturate((x - edge0) / (edge1 - edge0));
        x * x * (3.0 - 2.0 * x)
    }
}
```

---


# Testing GPU code

```rust
#[cfg(test)]
mod tests {
    use common::smoothstep;

    #[test]
    fn test_smoothstep() {
        assert_eq!(
            smoothstep(0.1, 0.5, 0.2),
            0.15625
        );
    }
}
```

---

# Tight Integration

```rust
#[derive(Copy, Clone, Pod, Zeroable)]
#[repr(C)]
pub struct ShaderConstants {
    pub width: u32,
    pub height: u32,
    pub time: f32,
}

#[spirv(fragment)]
pub fn main_fs(
    #[spirv(push_constant)] constants: &ShaderConstants,
    output: &mut Vec4
) {
    ...
}
```
---

# SPIR-V

* Intermediate representation for shader code
* Vulkan and OpenCL targets
* Open source toolchain!
* Supports all modern graphics APIs, *with cross compilation*
  [https://github.com/KhronosGroup/SPIRV-Cross](https://github.com/KhronosGroup/SPIRV-Cross)
  [https://github.com/grovesNL/spirv_cross](https://github.com/grovesNL/spirv_cross)

---

# SPIR-V Cross Compilation

![width:720px](https://www.khronos.org/assets/uploads/apis/2018-spir-api-ecosystem.jpg "Attribution: Khronos Group")

---

# Optimization

* Optimizing compilers are present, but typically vendored in kernel-space drivers
* SPIR-V can be optimized in user space
* Optimizations in shader code have been laborious to test
* Rust code can be benchmarked and tested on the CPU
* Libraries of pre-optimized code can be used with 3rd party crates (soon)

---

# See Also...

* [Rust Cuda](https://github.com/Rust-GPU/Rust-CUDA)
* [wgpu](https://github.com/gfx-rs/wgpu)
* [Ash](https://github.com/ash-rs/ash)
