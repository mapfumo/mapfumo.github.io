---
title: "You Can't Escape Rust: Rust for Blockchain"
date: 2024-11-04T07:29:49+10:00
draft: false
cover:
  image: /img/blkchain-photo.jpg
  alt: "Antony mapfumo - Rust for Blockchain"
  caption: "Artifacts are software applications, user interfaces, prototypes, Prototypes, etc."
tags: ["blockchain"]
categories: ["Blockchain"]
---

I thought I could “get away with it”—avoiding Rust, that is. A few years back, I started trying to learn Rust, but honestly, it was for all the wrong reasons. I thought it would just be "cool to know." As it turns out, that’s not exactly the best motivator when dealing with a language known for a steep learning curve and a strict compiler.
Fast forward to 2024, and my career path has taken me deeper into blockchain development. Naturally, Solidity dominates on the Ethereum network, but anyone who’s built on Ethereum knows the pain of high gas fees and relatively long block times. That got me exploring alternatives, and that’s when I came across the Solana blockchain. Rust is the main language for Solana, so here I am again, facing off with Rust. Sure, you can also write Solana programs (smart contracts in the Ethereum world) in Python, C, or C++, but Rust is the gold standard here.
Last time, I tried to brute-force my way through learning Rust and got discouraged, particularly by the borrow checker. But now I’m approaching it differently, taking my time and following what works. Here’s what I’ve found to be the best way to learn Rust:

### The Best Way to Learn Rust

If you’re serious about learning Rust, skip the shallow dives. Rust requires focus and patience, but if you approach it the right way, it’s incredibly rewarding. Here’s the path I’d recommend:

1. Start with [The Rust Programming Language](https://doc.rust-lang.org/book/) Book (aka The Book): This is the foundational Rust guide, covering essentials like ownership, borrowing, and the borrow checker. It might feel dense, but it’s worth it.
2. Get Hands-On with [Rustlings](https://github.com/rust-lang/rustlings): The Rustlings exercises are quick drills that teach the core concepts of Rust. They’ll give you the hang of the borrow checker, mutability, and error handling while letting you practice code that the compiler will guide you on fixing.
3. Write Small Programs: Start with simple projects—a to-do list app, a basic HTTP server, a calculator. This keeps the focus on learning Rust syntax and Rust’s way of managing memory without overwhelming you with complexity.
4. Join the Community: Rust has an active community. Whether it’s the [Rust subreddit](https://www.reddit.com/r/rust/), Discord channels, or the Rust users forum, this is where you’ll find the encouragement and insight to push through the tricky parts.
5. Tackle a Real Project Relevant to You: Once you’re comfortable with the basics, dive into a project that matters to you. For me, that’s blockchain development. I intend to eventually write a [simple blockchain](https://github.com/mapfumo/golang-blockchain) I wrote in GoLang in Rust in addition to continuously write web3 and dApps (Decentralised Applications) on the Solona blockchain in Rust. When you apply Rust to something that aligns with your goals, the motivation stays high.

### Common Rust Gotchas

Rust has its quirks—things you’ll trip over until you’re used to its rules. Here are a few things to watch out for:

- Borrow Checker Frustrations: Rust’s borrow checker can feel like a stern teacher. It enforces strict ownership rules, and if you’re used to languages that handle memory more loosely, this will be an adjustment. The key? Embrace it. Rust’s rules aren’t there to be an obstacle—they’re to help you write code that doesn’t leak or crash.
- Option and Result Types: Rust doesn’t have null values. Instead, you’ll use Option and Result to handle cases where a value might be absent or an operation could fail. These types feel strange at first, but they’re a game-changer for writing reliable code.
- Moving vs. Cloning: When you pass ownership of a variable, it “moves,” and you can no longer use it in the original spot. You might end up using .clone() to sidestep this, but be careful—cloning can get expensive. Learning when to clone and when not to clone is part of mastering Rust’s ownership model.

### Rust and Blockchain

Rust’s memory safety and performance make it a natural fit for blockchain development. Blockchain is about consensus, integrity, and immutability—three areas where Rust shines. Its strict compiler ensures code reliability and security, two critical elements in blockchain development. Plus, Rust’s low-level capabilities let you write high-performance code, essential for the demands of distributed networks and cryptographic operations.
In Ethereum, Solidity and the Ethereum Virtual Machine (EVM) set the standard, but for Solana, Rust is the primary tool. If you’re getting into blockchain and want to work with Layer 1s like Solana or Polkadot, Rust is practically required.

### Solana and Anchor

The Solana blockchain leverages a unique runtime and architecture, meaning writing smart contracts (called “programs” in Solana) here is quite different from Ethereum. Solana uses a framework called **[Anchor](https://www.anchor-lang.com/)**, which simplifies the process of building programs in Rust. Anchor provides a set of macros that streamline things like error handling, account validation, and deserialization—major pain points when developing directly on Solana.
With Anchor, you can write Solana programs faster and with fewer errors, thanks to built-in safeguards. If you’re serious about Rust and blockchain, Anchor is a tool worth learning. It’s not just an easier way to work with Solana; it also gives you a head start on mastering Rust in a blockchain context.

### Conclusion

Rust isn’t a language you “pick up.” It’s a language you “commit to.” This time around, I’m learning it with intention—applying it to real-world blockchain projects and taking my time to master the foundations. Rust’s strictness and complexity are there for a reason. Once you get past the initial hurdles, you’ll realize it’s a powerful ally in building software that’s fast, safe, and reliable.
