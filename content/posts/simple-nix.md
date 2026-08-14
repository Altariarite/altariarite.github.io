+++
title = "Nix made simple"
date = 2026-08-14
draft = true
+++

You do not need to learn NixOS, Flakes, or Home Manager to benefit from Nix. It actually feels not so different from `brew` or `pip`.

If you are getting started with Nix (the package manager, like Homebrew), or NixOS (the Linux distribution, like Ubuntu), there's a daunting amount of information online. All sorts of fancy stuff to use. I started by reading a [book](https://nixos-and-flakes.thiscute.world/) on NixOS and Flakes and Home Manager (oh my!). I copied things to my config without really understanding what they are doing.

While it's a nice book, recently^[1] I realised you can start super simple: 

- by using Nix just like any other package manager, but with nicer properties
- by writing a simple file to declare the packages. It's like `requirements.txt`, `package.json`, `Cargo.toml`...You name it
  
And that's really all you need to start using Nix and get all the benefits.

## Replacing homebrew

Mac users will be familiar with `brew install`.

