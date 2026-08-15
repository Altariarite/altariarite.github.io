+++
title = "Nix made simple"
date = 2026-08-14
draft = true
+++

> Verdict: You do not need to learn NixOS, Flakes, or Home Manager to benefit from Nix. It actually feels not so different from `brew` or `pip`.

If you are getting started with Nix (the package manager, like Homebrew), or NixOS (the Linux distribution, like Ubuntu), there's a daunting amount of information online. All sorts of fancy stuff to use. I started by reading a [book](https://nixos-and-flakes.thiscute.world/) on NixOS and Flakes and Home Manager (oh my!). I copied things to my config without really understanding what they are doing.

While it's a nice book, recently[^1] I realised you can start super simple: 

[^1]: After a conversation with [trofi](https://trofi.github.io/), thank you!

- by using Nix just like any other package manager, but with nicer properties
- by writing a simple file to declare the packages. It's like `requirements.txt`, `package.json`, `Cargo.toml`...You name it
  
And that's really all you need to start using Nix.

## Level 1: Replacing homebrew

Mac users will be familiar with `brew install`. You can do the same with `nix-env -i`:
```
❯ nix-env -i hello

installing 'hello-2.12.3'
this path will be fetched (30.3 KiB download, 110.4 KiB unpacked):
  /nix/store/xcb6pmfxwh0x7xn0qhcfhw9b4wv53fcm-hello-2.12.3
copying path '/nix/store/xcb6pmfxwh0x7xn0qhcfhw9b4wv53fcm-hello-2.12.3' from 'https://cache.nixos.org'...
building '/nix/store/pxi4fgnqdnf73miqlx7zyfwciv5dkyh3-user-environment.drv'...
```
## Level 2: Going Declarative




Well why should you do all the above instead of using `brew install`? 

## Benefit 1: Nix tidies your dependencies
This becomes obvious with programs that depends on other programs. Take clojure for example, the official doc says:
> ## Prerequisite installation details
> ...
> ### Java
> Clojure requires Java. ....
> If you don’t already have Java installed, we recommend installing Adoptium Temurin 25.
> To use the Adoptium Temurin installers:
>
>   Go to https://adoptium.net/
>
>   Download and run the installer appropriate to your platform
>
>  Ensure java is on the system PATH

If you `nix-env -i clojure`, you don't have to any of that. My clojure just runs after install:
```
❯ clojure
Clojure 1.12.5
user=>
```
You can actually see all the packages clojure depend on:
```
❯ nix-store -q --tree `which clojure`
/nix/store/daxm53q78r1vq0a3qpgwj0vbs494y7hz-clojure-1.12.5.1664
├───/nix/store/9gbwccpqidljx6rlhknv2nbm25b9jshg-zulu-ca-jdk-21.0.11
│   ├───/nix/store/4nhi70iyzcjabbgm2hm63h6hiqjcwcjr-set-java-classpath-hook
│   └───/nix/store/9gbwccpqidljx6rlhknv2nbm25b9jshg-zulu-ca-jdk-21.0.11 [...]
├ ...
└───/nix/store/daxm53q78r1vq0a3qpgwj0vbs494y7hz-clojure-1.12.5.1664 [...]
```
And the packages are isolated. You don't pollute your global path with java:
```
❯ nix-store -q `which java`
error: path '/usr/bin/java' is not in the Nix store
```
