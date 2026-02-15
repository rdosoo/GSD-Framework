# Build in Five Studio — Cursor Setup

## Quick Start

1. Create a new folder (or repo) called `build-in-five-studio`
2. Copy the `.cursorrules` file into the root of that folder
3. Open the folder in Cursor
4. Start a new chat (Cmd+L / Ctrl+L)
5. Describe your app — the AI will guide you through designing it

## How It Works

Cursor automatically reads `.cursorrules` from the project root and applies it as a system prompt to every conversation in that workspace. This means:

- Every new chat in this workspace behaves as the Build in Five Studio assistant
- No configuration needed — just open the folder and start talking
- The tech stack constraints are baked in — generated code always follows the standard

## For Non-Technical Users

Open Cursor, open this folder, and start a chat with something like:

- "I need an app that helps underwriters review exposure data"
- "I want a dashboard for tracking client onboarding progress"
- Or paste a detailed spec and the assistant will generate a brief immediately

The assistant will ask you questions, create an App Brief, and then generate the code scaffold — all within Cursor.

## For Developers

After the non-technical user has generated an App Brief and scaffold prompt:

1. Review the generated brief for completeness
2. Run the scaffold prompt to generate the application code
3. Run `docker-compose up --build` to test locally
4. Deploy via the Application Factory

## Sharing With Your Team

Option A: Share the folder (with .cursorrules) via your team's shared drive or repo
Option B: Each team member copies .cursorrules into their own workspace
Option C: Add .cursorrules to a shared Git repo that everyone clones

The .cursorrules file is the only file needed — everything else is generated.
