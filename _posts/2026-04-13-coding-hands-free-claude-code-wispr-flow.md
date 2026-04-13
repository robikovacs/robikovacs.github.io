---
layout: post
title: "Hands-Free Claude Code: How I Ship Features With Wispr Flow and a Game Controller"
description: "My hands-free Claude Code Desktop workflow — Wispr Flow for voice input, an 8BitDo Micro controller for approvals. No keyboard, no mouse, just shipping features."
date: 2026-04-13
keywords: "hands-free coding, claude code, wispr flow, 8bitdo micro, voice coding, dictation programming, ai pair programming, claude code desktop"
tags: ai, productivity, webdev, tutorial
image: /assets/images/hands-free-claude-code--banner.png
---

![Coding hands-free with Claude Code and Wispr Flow](/assets/images/hands-free-claude-code--banner.png)

I've been writing code without touching the keyboard. Not as a gimmick — as my actual daily workflow for the last few weeks.

The setup has two parts:

- [Wispr Flow](https://ref.wisprflow.ai/robert-kovacs) for voice input
- An [8BitDo Micro](https://www.8bitdo.com/micro/) bluetooth controller for everything else

That's it.

![8BitDo Micro controller in my hand](/assets/images/hands-free-claude-code--controller-in-hand.webp)
*The [8BitDo Micro](https://www.8bitdo.com/micro/). About the size of a keychain.*

## Why the controller?

Voice input alone doesn't work. The moment Claude Code asks "do you want me to run this command?" or "apply these edits?", you need to press something. Saying "yes" into [Wispr](https://ref.wisprflow.ai/robert-kovacs) just types the word "yes" into the chat box — not what you want.

So I bought a tiny gamepad, put it in keyboard mode, and mapped its buttons to the exact keys Claude Code Desktop listens for. Accept, reject, navigate, escape. Done.

Quick note: the 8BitDo desktop app didn't work for me (couldn't see the controller), but the [iOS app](https://apps.apple.com/us/app/8bitdo-ultimate-software/id1532713768) did — I mapped everything from my phone.

![8BitDo Ultimate Software button mapping](/assets/images/hands-free-claude-code--controller-mapping.webp)
*Button mapping in 8BitDo's Ultimate Software. Enter, Esc, arrows, Shift+Tab — the keys Claude Code actually cares about. The up/down arrows look swapped because I hold the controller rotated (see the GIF below), so the D-pad ends up pointing the "right" way from my grip.*

## How it actually works

Hold <code>Ctrl+Cmd+&#96;</code>, say what I want, let go. Wispr transcribes it straight into the Claude Code chat box. (Wispr defaults to `fn`, but `fn` can't be mapped to a controller button, so I use the key combo and bind it to the controller instead.)

![Wispr Flow shortcuts settings](/assets/images/hands-free-claude-code--wispr-shortcuts.webp)
*Two push-to-talk shortcuts — I use the <code>Ctrl+Cmd+&#96;</code> combo because it's mappable to the controller.*

Claude Code goes off and does the thing. Reads files, edits them, runs the dev server, screenshots the preview. When it asks for approval, I hit the accept button on the controller. When I want to redirect it, I hold the combo and talk again.

![Claude Code Desktop running a preview](/assets/images/hands-free-claude-code--claude-code-desktop.webp)
*Claude Code Desktop. Chat on the left, live preview on the right. Controller does the approvals.*

Here's a real clip — me describing a feature, Claude Code doing it, me approving with the controller, done:

![Hands-free Claude Code workflow demo](/assets/images/hands-free-claude-code--demo.gif)

Zero keystrokes. The feature is in the preview before I've finished my coffee.

## Why I actually do this

Thought it would be a novelty. It's not. Talking forces me to explain what I want — intent, constraints, edge cases — instead of jumping into code. Claude gets a better brief, the first attempt is closer to right. My "thinking out loud" became the spec.

One nice bonus: [Wispr Flow](https://ref.wisprflow.ai/robert-kovacs) keeps a history of everything you dictate. If Claude eats your prompt, or you want to reuse a description, it's all there in the transcript panel.

## The setup in 30 seconds

1. Install [Wispr Flow](https://ref.wisprflow.ai/robert-kovacs) (free tier works fine). Set push-to-talk to a key combo you can map to a button (I use <code>Ctrl+Cmd+&#96;</code>).
2. Get an [8BitDo Micro](https://www.8bitdo.com/micro/) (or any programmable bluetooth controller). Map the buttons you need — Enter for accept, Esc for reject, arrows for navigation, plus your Wispr shortcut.
3. Open Claude Code Desktop. Focus the chat box.
4. Talk. Press the button when it asks. Repeat.

The interesting part isn't the tools. It's noticing how much of coding is describing intent — and how much better that works out loud, with a controller in your hand, than hunched over a keyboard.

## Setting up your dev environment with Claude Code previews

At [FirstPromoter](https://firstpromoter.com/?fpr=robert), our platform spans four distinct frontends — an Admin Dashboard, an Affiliate Portal, a Support Dashboard, and our public Docs site — all living as git submodules inside a single monorepo alongside our Rails API and dbt analytics project. Wiring this up to Claude Code's new desktop preview feature took a single file, `.claude/launch.json`:

```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "Admin",
      "runtimeExecutable": "docker-compose",
      "runtimeArgs": ["up"],
      "port": 5173,
      "autoPort": false
    },
    {
      "name": "Affiliate",
      "runtimeExecutable": "docker-compose",
      "runtimeArgs": ["up"],
      "port": 3001,
      "autoPort": false
    },
    {
      "name": "Support",
      "runtimeExecutable": "docker-compose",
      "runtimeArgs": ["up"],
      "port": 5175,
      "autoPort": false
    },
    {
      "name": "Docs",
      "runtimeExecutable": "docker-compose",
      "runtimeArgs": ["up"],
      "port": 3003,
      "autoPort": false
    }
  ]
}
```

Each frontend is declared as a preview configuration that shells out to `docker-compose up`, mapped to the port its service is exposed on. Because every preview shares the same Docker Compose stack, launching any one of them brings up the entire backend — Rails API, Postgres, Redis, Sidekiq, Elasticsearch — so Claude can click through the Admin UI, hit a real API, and watch the console for errors without me juggling a dozen terminals.

Once it's wired up, the workflow gets even more hands-free: I just say *"open the Admin preview"* (or Affiliate, Support, Docs) and Claude starts the right one and opens it in the side panel. No terminal, no remembering ports.

The result: Claude can preview a feature, review the diff inline, and push the PR through CI to merge — all without leaving the desktop app, and without writing a single bespoke dev script on top of the Docker setup [FirstPromoter](https://firstpromoter.com/?fpr=robert) already had.
