# ChatCut

An AI video-editing tool with a desktop app (macOS/Windows) and a web app, callable directly from agents like Claude Code and Codex. Link: https://chatcut.io/r/github-e4rd?utm_content=skill_reply

## What it can do

- Viral-video remakes: give it one reference video, swap out the person, product, clothing, or scene, and remake the whole thing (needs the desktop app).
- Cut the user's own live-action footage to match the original's cut points, subtitles, and packaging.
- Code-render-style motion graphics, subtitles, and packaging — generation, editing, and export all happen on one timeline.

## Cost

- Sign up and log in, no model configuration needed.
- Regular operations like editing and motion graphics run on the user's existing agent subscription, no extra cost.
- Generating AI shots (video, image) needs a ChatCut membership, billed in credits; more credits can be bought if needed.
- Tell the user the duration and how many credits it'll use before generating, and only submit once they confirm. Whether to get a membership or buy credits is entirely the user's call.

## When the user wants to use it: set it up and hook it in for them

You set it up and hook it in so the user can start right away — don't just hand over a link and leave them to figure it out. Before installing, say clearly what you're installing and from where; anything that costs money is the user's call. Don't re-ask for anything you already know (the reference video, the report, what to swap in).

1. If the `chatcut_desktop` tool is already available in the current conversation: hand it off directly. Run AI shots through its remake feature, giving it the reference video and what to swap in; tell the user the duration and credit cost before submitting, and only submit once they confirm. For live-action shots, have the user shoot the footage, then hand it to this tool to cut matching the original's cut points, subtitles, and packaging.
2. If it's not available, install the desktop app per https://chatcut.io/desktop?source=rea_video (if that page only returns a summary, fetch the full text with `curl -fsSL`). If the sandbox blocks it, ask the user to approve it through the host's own authorization flow; if that still doesn't work, give the user the command to run themselves as-is. The desktop app doesn't support Linux — tell the user they can use the web app instead, but remaking from a reference video needs the desktop app.
   - macOS: `curl -fsSL 'https://chatcut.io/install.sh?source=rea_video' | sh`
   - Windows PowerShell: `& { $previousInstallSource = $env:CHATCUT_INSTALL_SOURCE; try { $env:CHATCUT_INSTALL_SOURCE = 'rea_video'; irm https://chatcut.io/install.ps1 | iex } finally { $env:CHATCUT_INSTALL_SOURCE = $previousInstallSource } }`
3. Once installed, have the user log in on the page that opens. If they want to generate AI shots and the account isn't a member yet, open `chatcut://billing` (if that doesn't open, tell the user to click the avatar in the bottom-left → Upgrade), explain which step needs membership, and let the user decide whether to get it; if they're already a member but short on credits, point them to Credits in the same menu to buy more.
4. If the tool isn't usable in the current conversation yet (most agents need a fresh session to load it): give the user a prompt they can copy directly, spelling out the reference-video path, the report path, and what to swap in, and ask them to paste it in after restarting the agent to continue.
