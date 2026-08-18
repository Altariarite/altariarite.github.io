+++
title = "Nix made simple"
date = 2026-08-14
+++

> Verdict: You do not need to learn NixOS, Flakes, or Home Manager to benefit from Nix. It actually feels not so different from `brew` or `pip`.

Nix is a nice package manager that can be used on Mac and Linux. Recently[^1] I realised you can start super simple: 

[^1]: After a conversation with [trofi](https://trofi.github.io/), thank you!

- by using Nix just like any other package manager, but with supercharged properties
- by writing a simple file to declare the packages. It will be like `requirements.txt`, `package.json`, `Cargo.toml`...

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
While the above is convenient, it's not the true power of Nix. Nix really shines when used declaratively.
## Level 2: Going Declarative
We can say want packages we want in a `package.nix` file:
```nix
with import <nixpkgs> { };
[
  vim
  git
  # Keep the rest of your packages...
]
```
And when we run `nix-env -f package.nix -ir`, Nix installs the packages, and removes all packages that are not in the list. Now we have an environment that's always matching what's written down in the file.

## Level 3: Pinning package versions
So far, package names have seemed to come out of thin air. Where do they come from, and how does Nix decide which versions to install?

`<nixpkgs>` refers to a copy of the Nix Packages collection, usually provided by a Nix channel. I will skip the details of channels here because we are getting rid of them. People think they are bad because they can change under your nose, which means the same `package.nix` may select different package versions over time. Not so declarative.

We can make the configuration truly reproducible by pinning nixpkgs to a specific revision with [npins](https://github.com/andir/npins):
```
cd "$HOME/.config/nix" # The directory of your package.nix file
npins init --bare
npins add github NixOS nixpkgs --branch nixpkgs-unstable
```
This creates an npins directory containing the pinned revision. Update `package.nix` to use it:
```nix
let
  sources = import ./npins;
  pkgs = import sources.nixpkgs { };
in

with pkgs; [
  vim
  ripgrep
]
```
Installing the packages from the pinned snapshot is the same:
```
nix-env -f package.nix -ir
```
The pin stays fixed until you explicitly update it:
```
npins update nixpkgs
```
What's so cool about Nix?

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
You can take this a step further and declare different versions of clojure and java for each individual project in something called `shell.nix`, but that can be another blog post.

## Benefit 2: The time machine
Have you ever installed something and it broke your environment? During college I have spent a semester helping students install Haskell environment, and it was not fun when you mess up and try to untangle packages. Sometimes you wish to delete the whole thing and start over.

Nix is fantastic at dealing with this because it's like a time machine.

Whenever nix-env changes your user environment, Nix keeps the previous version as a generation. A generation is a snapshot of the packages that were installed at that point in time.

For example, suppose we update our pinned packages and rebuild:
```
npins update nixpkgs
nix-env -f package.nix -ir
```
If the update breaks something, we can immediately return to the previous environment:
```
nix-env --rollback
```
And just like that we restore our environment to the previous version.

We can see all available generations with:
```
nix-env --list-generations
```
The output looks roughly like this:
```
  12   2026-08-15 10:32:41
  13   2026-08-17 18:06:12
  14   2026-08-18 09:45:27   (current)
```
We can also jump directly to a particular generation:
```
nix-env --switch-generation 12
```
Switching generations is fast because Nix does not reinstall everything. Each generation points to packages already stored in /nix/store, and
Nix atomically changes which generation is active.

Nix gives me peace of mind and now I can't go back to using package manager that's not declarative. 
