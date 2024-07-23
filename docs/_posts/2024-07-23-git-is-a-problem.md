---
layout: post
title: "The Git Ecosystem Might Become a Problem."
date:  2024-07-23
---

# Introduction
Version control is an integral part of the software development ecosystem. For developers to effectively collaborate, it needs to "stay out of the way". Time spent struggling with version control is time that's not spent writing code, thinking about code, or adding value to your project in any way. The *first* forms of version control were extremely rudimentary, and even when they did exist, many people didn't use them, and when they did, it involved a lot of added work. Some just preferred the shouting kind of version control:

> I had witnessed students in earlier courses do the shouting kind of version control: the students shared a source tree over NFS and shouted "I'm editing this file" when they were changing something. This did not seem like an effective method to me [^1]

## Before Git
Many developers found existing methods of version control cumbersome and slow. When Linus Torvalds first created Git, one of his core design principals was "WWCVSND", or "What Would CVS Not Do?[^2]" He felt that CVS was flawed at an architectural level, and so solutions like Subversion could never work out, because they suffered from the same architectural flaws[^3].

One core principal of CVS was that there's a centralized repository for all project files and version history. This had a few problems. 
- If the server went down, the entire project was inaccessible
- Managing differences between repositories was fundamentally a second class feature. This meant that merging and branching were complex, buggy interactions.
- Upstreaming changes was discouraged simply because of the complexity.
[^4]

For projects like the Linux kernel, this meant putting a lot of stress on the entity managing that repository. It required a single person/group to coordinate *every contributor*. As project scaled, CVS didn't easily allow the distribution of work.

That's where BitKeeper comes in. BitKeeper started with the goal to "Figure out a means by which Linus can surround himself with some number
of people who do part of his job.[^5]" It was developed with distribution in mind (everybody gets a repository). This meant that everyone had their own "line of development", and so they could work on changes locally then add them to the project when they were ready, without worrying about the changes everyone else was making to the project. 

Linus Torvalds credits BitKeeper for the core design ideas behind Git, describing it as the "the first version control system I ever felt was worth using at all.[^5]" However, when he first tried to use BitKeeper, there was massive internal backlash. BitKeeper was closed source, and was developed as part of a commercial product. Developers felt that this was meant BitKeeper couldn't be trusted for sustained open source use, as they could revoke the free license at any time.

BitKeeper continued to be used for the Linux kernel for some time, until one of the kernel developers violated BitKeeper's license and reverse engineered it to "pull stuff out of bk trees without agreeing to the bk license.[^6]" At this time, the kernel project began looking at alternatives to BitKeeper. This was a problem the industry at a whole was looking to address.

## Darcs/Patch Algebra
At the time, there were several alternatives being developed, and the version control space was rapidly changing. The GNU project was working to develop a version control system called GNU arch, and out of that evolved a project called Darcs. The theory behind Darcs was very different from prior attempts to design a version control system, and it was called *patch algebra*. At a high level, patch algebra was developed by David Roundy. David Roundy wasn't not a computer scientist, and his main area of expertise wasn't even around computer science. He was a physicist, and so he approached the problem from the perspective of a physicist. One could generalize physics about the study of interaction and change in the material universe, and so it's no surprise that patch algebra is all about *changes*[^7]. It represented those changes as patches, and a patch could be a variety of things, from the creation or deletion of a file, to changing or moving a file. While the idea held promise, it had a few issues. One of the last maintainers of Darcs, Florent Becker, stated that "The algorithm was not proper. It wasn't an actual algorithm because there was a bunch of holes in it, and this explained why it was so slow in dealing with conflicts.[^8]" Darcs never saw real adoption because of this. It was exponentially slow, and because the algorithm wasn't functionally complete, it lead to bugs that would cause massive issues like missing or incorrectly patched code.

Darcs, alongside other version control systems held potential, but that wasn't enough for the Linux project to commit (no pun intended) to any of them. And so, Linus created Git. Git introduced a lot of important ideas, representing changes as commits, robust branches, forks, and more. It was developed with simplicity first, rather than trying to store data in a conventional database table, he directly stored that data on the filesystem. This greatly simplified complexity, because that you could represent a filesystem, with a filesystem. Ideas like hash based references were also important contributions, it meant that you could de-duplicate data, and it provided a built in form of integrity check.

## GitHub
Git rapidly gained adoption, and today 93% of developers reported using Git for version control. This is in no small part due to the creation of GitHub. GitHub was founded in 2008 as the first major cloud hosting platform for Git. Within a year, GitHub proved to be a massive success, accumulating more than 46,000 public repositories[^10]. The platform rapidly scaled, and is still growing today. As it's grown, it's expanded in scope and functionality. From a simple place to host version control, today it offers a massive amount of services including (but not limited to):
- Issue tracking/project management: Issues, Pull Requests, Projects, and more
- Package hosting/management: Github Packages (most notably used by the Homebrew package manager), and Releases.
- Static site hosting: Github Pages
- CI: Github Actions
- Developer environments: Codespaces
- Programming LLM: Github Copilot
- Dependency tracking: Dependabot
- (Unofficially) an index for package managers: See [Rust](https://github.com/rust-lang/crates.io-index), [Nix](https://github.com/NixOS/nixpkgs), and [Homebrew](https://github.com/Homebrew/homebrew-core/pulls).

# The Problem
Github (and therefore git) has grown so ubiquitous that for many open source projects, existing on Github is the only real option. Contributors don't want to contribute to open source projects if they aren't on Github. The maintainer of the only GNU project hosted on Github noted that "since I moved it to GitHub the number of external contributors has gone from zero to several highly motivated people.[^11]" They still receive angry emails from Richard Stallman telling him that he's evil, but the motivations behind hosting your software on Github are hard to ignore.

Even Github alternatives like Gitea and Gitness are often hosted on Github, simply because it's how developers want to contribute. While Github alternatives do exist, their main selling points are usually compatibility *with* Github. For example, Gitness was developed by Harness, and Harness's *main product* is drone.io. One major selling point pushed by Gitness? *Compatibility with Github Actions.*

If you follow the development behind products like Gitea and Gitness, you notice a pattern. Most development work isn't around *making new features*, it's around *reaching parity*.

Even beyond the community buy in to *Github*, it also means that they're buying in to *git*. Alternative version control systems are rare, and niche. The Firefox project is working to migrate away from Mercurial, to Git. Usage of alternatives is going down, not up. Git isn't great, and there's still significant room to improve the version control ecosystem, but it's hard to switch to alternatives when switching to alternatives usually means moving away from *Github* too. 

While Github has contributed an immense amount to the ecosystem, and they've built an incredible product, the software development community needs to be wary. The software world *depends* on Github. While they have not abused that dependency to an extreme level, we're still relying on a for profit entity to always make the best choice for the community, and put profits second.

# Solutions?
If you'll recall, earlier in the article, I brought up Darcs, and patch theory. Git doesn't model conflict and change very elegantly (if you've ever fixed a merge conflict, you'll understand). Darcs attempted to solve this problem, but the algorithm used for patching was heavily flawed. And so, the last maintainer of Darcs, along with some other researchers started over. They developed a fundamentally new algorithm, one that addressed the previous issues with Darcs. That project is still active today as [Pijul](https://nest.pijul.com/pijul/pijul), however the community is small, and it sees little real world use. It's a difficult problem. Pijul solves a lot of the issues with git, and in many ways, is an objectively better product. However, using Pijul means that you, and all of the developers you work with need to learn a new system, one that isn't compatible with the larger ecosystem around git (namely, Github), so you're left using products that aren't as robust as the git alternatives.

One way this issue could be addressed would be by Github supporting alternate version control systems. However, this isn't realistic. It's asking a lot of Github, and from their perspective, it doesn't make much sense.

One way to *reduce* this cost is to use version control systems *compatible* with git. One such option, [Jujitsu](https://github.com/martinvonz/jj) offers many of the same advances over git, including patch algebra and the ability to undo git operations. This means you can take advantage of the ecosystem Git provides, while still having some advantages over using vanilla git.

To summarize, Git is far from perfect, but it's good enough that making improvements is difficult. As for the buy in to Github, you should consider the single responsibility principal in software design. Github performs the work of many different tools, and so options that provide alternatives to that functionality should be encouraged. Some examples include:
- Graphite, Jira - Issue management
- drone.io, Jenkins - CI
- Vercel - Site hosting

These are all options that better uphold the *single responsibility* principle.

[^1]: https://lwn.net/Articles/928581/
[^2]: https://www.youtube.com/watch?v=4XpnKHJAok8
[^3]: https://graphite.dev/blog/bitkeeper-linux-story-of-git-creation
[^4]: https://en.wikipedia.org/wiki/Concurrent_Versions_System
[^5]: https://lkml.org/lkml/1998/9/30/122
[^6]: https://www.linux.com/news/bitkeeper-and-linux-end-road/
[^7]: https://www.cs.tufts.edu/comp/150GIT/archive/david-roundy/theory-patches-2009.pdf
[^8]: https://the-stack-overflow-podcast.simplecast.com/episodes/for-those-who-just-dont-git-it/transcript
[^9]: https://stackoverflow.blog/2023/01/09/beyond-git-the-other-version-control-systems-developers-use/
[^10]: https://en.wikipedia.org/wiki/GitHub
[^11]: https://lobste.rs/s/fkzcja/ditching_github#c_ie3hfq
