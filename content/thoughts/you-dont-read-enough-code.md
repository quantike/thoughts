+++
date = '2025-01-23T08:55:33-06:00'
draft = false
title = 'You Dont Read Enough Code'
description = 'One path to being a better software engineer'
tldr = 'Read all the code that you can, using well-known libraries and applications as examples for your own work. Nothing is new, use this to your advantage.'
tags = ['opinions']
+++
You probably don't read enough code. I know I don't.

Almost everything we do in software is not new. The sum product of the business might be new to the market, but it’s powered by databases and HTTP just like every other start up. This is to your advantage. 

Almost everything has been done before. Your job is to read it, understand it, and iterate on it. 

> This post is inspired by the xweet from [@kuberdenis](https://x.com/kuberdenis) titled "You are not a software engineer."[^1] This post serves as one way to overcome the challenge he describes.

[^1]: [You are note a software engineer.](https://x.com/i/bookmarks?post_id=1869383856723464402) 

## What reading more code gives you

Programmers who are "well-read" tend to:

- Be able to estimate the effort for a new feature or service more effectively
- Know the ins-and-outs of the software they work with
- Diagnose problems and craft solutions quickly

You'll want to read more code to be a better engineer no matter your level. Doing so will set you apart from your peers who read less code than you. 

## Getting started

First, find a repository of interest. This can be a project from work, or a public project on GitHub, but you need a repository you find interesting. If you're lost, pick a repo for a tool you use often. If you're really lost, here's a few I like: 
- Bybit Simple Market Maker[^2]
- Atuin ✨ Magical shell history[^3]
- SlateDb[^4]

[^2]: [`bybit-smm`](https://github.com/beatzxbt/bybit-smm) is a "simple" market maker from the X/Twitter personality [@BeatzXBT](https://x.com/BeatzXBT) which does some fairly complex market making strategies on live market data, in Python nonetheless.
[^3]: [`atuin`](https://github.com/atuinsh/atuin) is (1) an excellent shell plugin, and (2) a fascinating read on how to build a performant, CLI tool. Authored by [Ellie Huxtable](https://ellie.wtf/).
[^4]: [`slatedb`](https://github.com/slatedb/slatedb) is a cloud native embedded storage build on object storage.

Now that you have a codebase, there are two ways to approach it (if you're a fan of tree traversals[^5]):

1. Depth-first search (DFS).
2. Breadth-first search (BFS). 

[^5]: [Tree traversal](https://en.wikipedia.org/wiki/Tree_traversal)

## Grasp the high level concepts

We need to develop the 30,000ft view of the repository. This calls for **breadth-first search**. Start by at the root and explore the subdirectories before going deeper. I use BFS to find the "meat" of the code. Taking `slatedb` as an example, I'd locate the `Db` struct.

```rust
// From `https://github.com/slatedb/slatedb/blob/main/src/db.rs`
pub struct Db {
    pub(crate) inner: Arc<DbInner>,
    /// The handle for the flush thread.
    wal_flush_task: Mutex<Option<tokio::task::JoinHandle<Result<(), SlateDBError>>>>,
    memtable_flush_task: Mutex<Option<tokio::task::JoinHandle<Result<(), SlateDBError>>>>,
    write_task: Mutex<Option<tokio::task::JoinHandle<Result<(), SlateDBError>>>>,
    compactor: Mutex<Option<Compactor>>,
    garbage_collector: Mutex<Option<GarbageCollector>>,
}
```

Clearly, for a database, something like `Db` is a load-bearing strucure. Let's dig in.

## Dive in to the details

Now that we've BFS'ed into a critical component, I like to switch to **depth-first search**. Following that pattern, we might go look at `DbInner` next

```rust
// From `https://github.com/slatedb/slatedb/blob/main/src/db.rs`
pub(crate) struct DbInner {
    pub(crate) state: Arc<RwLock<DbState>>,
    pub(crate) options: DbOptions,
    pub(crate) table_store: Arc<TableStore>,
    pub(crate) wal_flush_notifier: UnboundedSender<WalFlushMsg>,
    pub(crate) memtable_flush_notifier: UnboundedSender<MemtableFlushMsg>,
    pub(crate) write_notifier: UnboundedSender<WriteBatchMsg>,
    pub(crate) db_stats: Arc<DbStats>,
    pub(crate) mono_clock: Arc<MonotonicClock>,
}
```

Now the choice is yours, go read about `DbState`, `DbOptions`, etc. until your face is blue. But remember the objective, you should be able to return to `Db` with a **greater understanding** than when you last looked at it. You should know how the database manages state, how to configure it, what a `TableStore` is, etc. Going deep should inform you higher-level thinking about the code.

{{< callout type="tip" text="Sometimes reading code isn't enough. I recommend reading the Pull Request histories that affect the code you are looking at. Go see the last time `Db` was updated, and read what the motivations were, what what the state beforehand, etc." >}}

The higher-level reasoning develops as you build understanding with the code base. Design decisions reveal themselves, and so do the short-comings. 

{{< callout type="tip" text="To better understand short-comings, go read the Issues." >}}

## Read Efficiently

Use code navigation tools:

- Check the Github side bar for their "Symbols" section. Use it to navigate. Click on the struct you are looking at and go find how it's implemented and where it is used.
- When reading code in an IDE get familiar with "go to definition". Make a hot-key for it (see: [my neovim config](https://github.com/quantike/dotfiles)). 

Use code understanding tools:

- RTFM![^6]
- I find that AI tools work best at a narrowly scoped coding problem

[^6]: [RTFM](https://en.wikipedia.org/wiki/RTFM#:~:text=RTFM%20is%20an%20initialism%20and,forum%2C%20software%20documentation%20or%20FAQ.)

**Example**: "What does the `|= 1 << i` syntax do in this `.hash()` function for my locality-sensitive hashing implementation?"[^7]

```rust
fn hash(&self, vector: &Array1<f64>) -> u64 {
	let mut hash_value: u64 = 0;

	for (i, hyperplane) in self.hyperplanes.iter().enumerate() {
		let dot_product = vector.dot(hyperplane);
		if dot_product > 0.0 {
			hash_value |= 1 << i;
		}
	}

	hash_value
}
```

[^7]: [Locality sensitive hashing](https://en.wikipedia.org/wiki/Locality-sensitive_hashing) 

## The habits you need to change
Be a voracious reader. Read the deploy scripts. Read the config files. Read the Dockerfiles. You should read every piece of the code in some capacity (maybe not the lockfiles though, unless you’re working on [bun](https://github.com/oven-sh/bun) or [uv](https://github.com/astral-sh/uv))

Challenge yourself to understand the tools you use. Very few developers know how their favorite tools work under the hood. Be different. Next time you reach for [fastapi](https://github.com/fastapi/fastapi), spend some time reading the code base.

Think about your own implementations. Has someone done this before? Is there a state of the art that can be referenced? Can I do it better?

Read more code. 
