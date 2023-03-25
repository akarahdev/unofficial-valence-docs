# Valence Overview
Valence is a framework for creating Minecraft servers built in Rust! Valence aims to be:
* **Complete**. Abstractions for the full breadth of the Minecraft protocol.
* **Flexible**. Can easily extend Valence from within user code. Direct access to the Minecraft protocol is provided.
* **Modular**. Pick and choose the components you need.
* **Intuitive**. An API that is easy to use and difficult to misuse. Extensive documentation and examples are important.
* **Efficient**. Optimal use of system resources with multiple CPU cores in mind. Valence uses very little memory and can support thousands of players at the same time without lag (assuming you have the bandwidth).
* **Up to date**. Targets the most recent stable version of Minecraft. Support for multiple versions at once is not planned. However, you can use a proxy with [ViaBackwards](https://www.spigotmc.org/resources/viabackwards.27448/) to achieve backwards compatibility with older clients.

However, you need to keep in mind that Valence is still in early development. This book may not be fully accurate and there will likely be lots of breaking changes to the API.

Do note that this documentation assumes you have read [The Book](https://doc.rust-lang.org/book/), have Rust installed via [rustup](https://rustup.rs/), and have the latest version of [Minecraft: Java Edition](https://www.minecraft.net/en-us/download).

## Running Examples
You're probably wondering what Valence is capable of. To see what it's capable of, you should load up an example. First off, clone the repository:
```
git clone https://github.com/valence-rs/valence.git
```

Next, you can type `cargo run --example` to see a list of examples. Then, you can run an example of your choice by running `cargo run --example <name of your example>`. We recommend running `parkour`, `conway`, `terrain`, and `cow_sphere`. To join the server, connect to `localhost` on the Direct Connect screen.

## Low-Level Control
Something to keep in mind is that Valence doesn't operate like you would interface a Spigot server with a plugin. Instead, Valence makes you create the server yourself, so plugins in the Spigot/Bukkit sense aren't necessary. This has it's advantages and disadvantages.

For one, you will be able to have fine grain control over your server. However, if you want vanilla mechanics, you'll have to implement them manually. This sounds overwhelming with such low-level control, but with Valence's abstractions it shouldn't be too bad to make servers without vanilla mechanics. It may take some getting used to though.

With that out of the way, let's move on to [Getting Started](./chapter_1.md)