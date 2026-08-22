# Becoming a Publisher

> These terms have not been reviewed by a lawyer and are not legal advice. They are here to set
> clear expectations between the registry and the people publishing through it.

The registry records **publishers**, not individual addins. Once you are listed, you host your own
addin list in your own repository and publish new versions whenever you like, without asking anyone.

That is the trade: you get to publish freely, and you own what you publish.

---

## What being listed means

You are listed by **identity** — the registry says "this is a real person or organisation, and this
is where their addins come from". It does not say your code has been read, tested, audited, or
approved. Nobody here reviews it. Users are told this plainly before their first install.

Being listed is not an endorsement, a certification, or a quality mark, and you should not describe
it as one.

---

## What you agree to

**You publish only what you have the right to publish.** Your own work, or work you have permission
and a licence to redistribute. If your addin is a fork, say so in its entry and honour the original
licence — including carrying its copyright and licence notice with every copy you distribute.

**And only what its author wants published here.** An open source licence gives you the right to
redistribute; it does not tell you the author wanted their work listed under someone else's name, in
a place they did not choose. Those are different questions, and the licence only answers the first.
If it is not yours, ask — and if the author would rather list it themselves, that is the better
outcome anyway: their entry, their release schedule, their name against it.

The registry enforces the narrow version of this mechanically — an addin's downloads must come from
the publisher's own GitHub account, so you cannot list software served from somebody else's. But do
not treat passing that check as having asked.

**You are responsible for your addins.** Clarion addins run *inside the IDE, in-process, with the
full privileges of the user running it*. An addin can read and write anything that user can. That is
a meaningful amount of trust, and it rests with you, not with the registry.

**You do not ship anything harmful.** No malware, no telemetry the user is not told about, no code
that reaches outside what the addin is described as doing. If your addin sends anything anywhere,
say so in its description.

**You keep your entries honest.** The version in your list matches the release it links to. The
description describes what the addin actually does. Download links point at your own releases and
are not repointed at something else later.

**You serve from your own account.** Download URLs must live under `github.com/<your-publisher-id>/`.
This is checked by the client, not by a person. It exists so that being listed once does not become
permission to serve arbitrary binaries from anywhere forever.

**You respect Identity uniqueness.** Clarion loads every subfolder of `accessory\addins` at startup
and refuses to start at all if two of them declare the same `<Identity name>` — the user sees
*"Identity name used by multiple addins"* and their IDE will not open. Do not publish an addin whose
Identity is already in use by another publisher's. Addin Finder checks and refuses to install rather
than break someone's IDE, but that check is there to catch honest mistakes, not to make name
squatting workable.

**You handle your own support.** Problems with your addin are reported to your repository. The
registry is not a support channel for your code, and issues raised here about it will be redirected.

**You respond to security problems.** If someone reports a vulnerability in your addin, act on it,
and publish a fixed version. If you can no longer maintain an addin, say so and ask to have it
delisted rather than leaving a known-broken version installable.

---

## Requirements

- Addins hosted in **public GitHub repositories** under your account or organisation
- **MIT** or another OSI-approved open source licence, with a `LICENSE` file in the repo
- Targets **.NET Framework 4.0-4.8** (`net40`-`net48`) — .NET 5+ will not load in Clarion
- A `README.md` and a `CHANGELOG.md`
- Releases published as GitHub releases, with the assets your entry links to

---

## What the registry does and does not do

**It does:** record who you are and where your list lives; make your addins discoverable; check that
download URLs belong to you; warn users before their first install about what addins can do.

**It does not:** review, test, build, sign, or scan your code; host your binaries; guarantee your
addin works or is safe; provide any warranty to users on your behalf; take responsibility for what
your addins do.

---

## Delisting

A publisher can be removed from the registry. The realistic reasons:

- publishing malware, or code that does something materially different from its description
- publishing someone else's work without the right to do so
- repeatedly claiming Identity names already in use by others
- an unfixed security problem that is not being addressed
- asking to be removed

Except where users are at active risk, you will be told first and given a reasonable chance to fix
the problem. If something is actively dangerous it comes out immediately and the conversation
happens afterwards.

**A `revoked` list is published alongside the registry.** Delisting stops new installs, but it does
nothing about copies already on people's machines — so revoked publishers are named, and Addin
Finder warns users who have their addins installed. Removal from the publisher list alone would be
silent, and silence is the wrong outcome for the one case where it matters.

Being delisted does not delete your repositories or affect your addins outside this registry.

---

## Applying

**Open an issue on this repository, from the GitHub account you want listed**, with:

- your GitHub account or organisation — this becomes your publisher `id`
- the repository that will hold your `addins.json`, and its default branch
  (`main` and `master` are both fine — the registry records which)
- the addins you intend to publish, with links to their repositories
- confirmation that you have read this document and agree to it

Have your `addins.json` in place before you ask, so the entry works the moment it is added. See
[msarson/clarion-addins](https://github.com/msarson/clarion-addins) for a working example.

The issue must come from the account being listed. GitHub authenticates whoever opens one, so that
is enough to show you control it. A request relayed some other way — by email, or by somebody on
your behalf — cannot show that, and will be asked to come back as an issue from the right account.

An issue rather than a pull request, deliberately. A PR against `registry.json` can change every
line in it, not only add a publisher, and merging is one click. Asking for the entry and making the
entry are different acts, and only one of them should be possible from outside.

Approval is a judgement call, not an automated one. It mostly comes down to whether the addins are
real, are yours, and do what they say.

---

## Changes to these terms

These terms may change as the registry grows. Material changes will be raised as an issue before
they take effect, so publishers can object or leave. The version of this document in effect is the
one on the default branch.

---

## A note on what this document is worth

This sets expectations and gives a stated basis for delisting someone. It does not transfer legal
liability, and no user should treat it as protection — an end user's real protection is the addin's
own licence, the fact that everything here is open source and readable, and being told honestly what
addins can do before installing one.

If you want terms with actual legal weight, they need a lawyer, not a markdown file.
