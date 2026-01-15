+++
title = 'On upgrading to Zig 0.14.0, language servers, and unstable software'
description = 'A deep dive into upgrading Zig to 0.14.0, debugging LSP issues with Mason and zls, understanding tarball naming conventions, and lessons about working with unstable software.'
subtitle = 'Zig on roadbumps'
# taxonomies = ["zig", "experiences"]
date = '2025-03-10T01:36:47-05:00'

[extra]
outdate_alert = true
+++

With the release of a [new version of Zig](https://ziglang.org/download/0.14.0/release-notes.html), I was interested to try some of the new features. As a MacOS user for my primary development (sorry Linux folks), this meant the usual steps.

```bash
brew upgrade zig
brew upgrade zls
```

Little did I know this would result in an 18h crash course in:

- Zig tarball naming conventions
- Language server registries
- Unstable software

Plus, a tidbit at the end about how to make this process easier.

## Everything broke

Ok so I ran my `brew update`s and quickly `nvim`'ed back into my current Zig project. Everything seems fine. I am typing something to the effect of

```zig
const std = @import("std");

// code...
try std.debug.print("Printing something", .{});
```

When I noticed I wasn't getting any code completions for the Zig standard library. Weird.

Ok so I restart my LSP (neovim, btw). Now I get this message:

```
LSP[zls][warning] zig standard library directory could not be resolved
```

Ok. Really weird.

## Ok maybe I need to update my LSP in Mason

I use [mason.nvim](https://github.com/williamboman/mason.nvim) for my language server management. Simple enough to update the LSP right? I do this all the time with rust-analyzer.

```
:MasonUpdate zls
```

It errored!?

```
e/nvim/lazy/mason.nvim/lua/mason-core/installer/init.lua:249: Installation failed for Package(name=zls) error='spawn: wget failed with exit code 8 and signal 0. \nFailed to download file "https://github.com/zigtools/zls/releases/download/0.14.0/zls-aarch64-macos.tar.xz"
```

Ok. Now like, super weird. There isn't a valid distribution for zls 0.14.0? I guess let's check the releases page on GitHub.

## Why isn't there a valid download file for `zls-aarch64-macos` on GitHub?

There _is_ a valid tarball for zls? When I check releases I see and assortment of

```
.../zls-{os}-{architecture}-0.14.0.tar.xz
```

which at first glance, _seems_ right. A closer inspection on the Mason error reveals that it's attempting to find a tarball that looks like

```
.../zls-{architecture}-{os}.0.14.0.tar.xz
```

which is wrong!

Amazing, we've located the error, this must be a simple fix now. Just go update the [mason-registry](https://github.com/mason-org/mason-registry), update Mason locally, then BOOM we're cooking with gas. Right. Right!?

Of course not.

## Your dependabot depends on faulty assumptions

So how does Mason know exactly what language server that you need? It has a language server registry, obviously (I did not know this but it just makes sense when you spend a few seconds thinking about it).

Ok so Mason has a registry. How does it know to update the registry? It has a dependabot of course!

Dependabots are pretty awesome, when they work. Like all software, when it works, you shouldn't even have to think about it, but when it doesn't work, it can be catastrophic[^1].

In this case, dependabot was not awesome. It was built on the assumption that all zls releases would follow the

```
.../zls-{architecture}-{os}.0.14.0.tar.xz
```

pattern, which the latest release of zls broke! This is, luckily a pretty easy [fix](https://github.com/mason-org/mason-registry/pull/9211) (you can read the diff here, but beware, spoilers ahead!).

[^1]: Catastrophic is totally overblown in this specific case, since this dependabot is for updating the registry link to a relatively niche piece of software that is by most definitions _not_ mission critical. I'd argue that the same principle of software failures apply though, since dependabots are updating software dependencies automatically in probably millions of repositorys across the internet.

## The tarball changed. Why?

I flocked to the Zig language Discord to find out. I was immediately told "you shouldn't need a language server, just use `zig build` and `zig test`". Much to my chagrin, I don't currently possess that level of programming enlightment, and I love my GruvBox syntax highlighting.

I did, however, get another piece of information from an engineer @ [Bun](https://bun.sh/)[^2], [Meghan](https://x.com/nektro?lang=en), who shed some light on _why_ the ordering had changed from `{architecture}-{os}` to `{os}-{architecture}`. She explained that zls changed the ordering to match Zig's ordering.

In Zig, the build system is pretty sweet. You can build for multiple targets to make a release directly in the `build.zig` file (this kind of thing is usually a headache). Here's neat example from the "[Handy Examples](https://ziglang.org/learn/build-system/#examples)" section on the Zig build system:

```zig
const std = @import("std");

const targets: []const std.Target.Query = &.{
    .{ .cpu_arch = .aarch64, .os_tag = .macos },
    .{ .cpu_arch = .aarch64, .os_tag = .linux },
    .{ .cpu_arch = .x86_64, .os_tag = .linux, .abi = .gnu },
    .{ .cpu_arch = .x86_64, .os_tag = .linux, .abi = .musl },
    .{ .cpu_arch = .x86_64, .os_tag = .windows },
};

pub fn build(b: *std.Build) !void {
    for (targets) |t| {
        const exe = b.addExecutable(.{
            .name = "hello",
            .root_source_file = b.path("hello.zig"),
            .target = b.resolveTargetQuery(t),
            .optimize = .ReleaseSafe,
        });

        const target_output = b.addInstallArtifact(exe, .{
            .dest_dir = .{
                .override = .{
                    .custom = try t.zigTriple(b.allocator),
                },
            },
        });

        b.getInstallStep().dependOn(&target_output.step);
    }
}
```

which can be ran like

```shell
$ zig build--summary all

Build Summary: 11/11 steps succeeded
install success
├─ install hello success
│  └─ zig build-exe hello ReleaseSafe aarch64-macos success 30s MaxRSS:249M
├─ install hello success
│  └─ zig build-exe hello ReleaseSafe aarch64-linux success 29s MaxRSS:255M
├─ install hello success
│  └─ zig build-exe hello ReleaseSafe x86_64-linux-gnu success 30s MaxRSS:228M
├─ install hello success
│  └─ zig build-exe hello ReleaseSafe x86_64-linux-musl success 30s MaxRSS:232M
└─ install hello success
   └─ zig build-exe hello ReleaseSafe x86_64-windows success 30s MaxRSS:253M
```

You can see that indeed, Zig does build in the `{architecture}-{os}` ordering! That's pretty cool. Zig is a nascent language, all things considered, and hasn't really centralized on lots of standards like some more developed languages have (or haven't in the case of Python). This kind of confusion is part of the "early adopter" experience.

[^2]: The people at Bun know their shit when it comes to Zig!

## Circling back to Mason

Let's check back in on Mason, and the aforementioned PR.

Oh. It's closed? Nice, I wonder why.

I decide to leave a comment, pointing out what I learned above. Figured I'd share the knowledge, and plus it's nice for maintainers of a language server registry to be aware that the language tooling is beginning to coalesce around a new standard. Zig history in the making!

Then I get the repsonse:

> "...you can check the tarballs on the 0.14.0 release they currently match the tarball naming scheme from 0.13.0"

Huh!?

So I check. They most certainly match the previous 0.13.0 release of `{os}-{architecture}` tarball naming. I thought we were finding consensus! I thought we had agreed on this new standard!?

## Software is about serving users, not standards

Sure enough, zls was back to 0.13.0's tarball naming convention. The tarballs updated at some point since I had gone to bed and before I checked in the morning.

I was a little dismayed that the naming convention hadn't stuck. But it's a valuable lesson about breaking changes in software. When you make a breaking change, it creates downstream issues. And in the case of zls, this downstream issue potentially effection hundreds (thousands?)[^3] of users.

As developers of software, we have to always focus on the fact that the software we write must first seek to serve the needs of a user. The "extra stuff" (standards, tooling, distribution, and so, so much more), is _also_ in service of the user! When we create standards, those standards should make it easier, not harder, to get the software into the hands of users.

We make these standards like [semantic versioning](https://semver.org/) to clearly communicate our intentions when distributed releases. Hey when `MAJOR` changes, expect breaking changes! Zig is still [pre-1.0.0](https://github.com/ziglang/zig/milestone/2), with a [zero-based versioning](https://0ver.org/), which means that every change to the `MINOR` can and will ship a breaking change!

So while zls was totally justified in making the breaking change in their tarball naming convention, they are also justified in reverting that change.

I can only guess why the tarballs were updated in the night to the old naming scheme, but from my point of view, it was a nice reminder to me about _why_ we write software.

[^3]: There are literally dozens of us... dozens! Of the Zig language server, zls, who are using Neovim, with Mason as their primary language server provider. On a real note, I have no real idea how many people may have actually been affected. If you have a good estimate, reach out, I'd love to know!

## A better way to manage Zig distributions

Along the way I learned quite a bit about language server registries, tarball naming conventions, and working with unstable software. I was impressed by the Zig community's deep knowledge and passion for the language. I also picked up a better way to manage my unstable Zig versions, [zigup](https://github.com/marler8997/zigup). I followed a quick tutorial and was able to get both version 0.13.0 and 0.14.0 to work on my Mac, yippee!

I did hit a little snag though, which may be a feature and not a bug. In Mason, you can't have multiple distributions of the same language server (to my knowledge, please correct me if this is wrong). This means that when I am writing in Zig 0.13.0 (some projects that haven't been migrated yet), I have to `:MasonUninstall zls` to uninstall zls for 0.13.0, then `zigup 0.14.0` to set my default Zig to 0.14.0, then `:MasonInstall zls` to install the latest 0.14.0 version of zls.
