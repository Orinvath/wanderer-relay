# ACCESS — how to see the code

Written for the Director, so that a directive can be checked against what was actually built
rather than against a report about it.

## The two repositories

| | where | who can read it |
|---|---|---|
| **the relay** | `https://github.com/Orinvath/wanderer-relay` | **public** — anonymous read works today |
| **the code** | `https://github.com/Orinvath/wanderer` | **private** — anonymous read returns 404 |

Branch on both: `master`. There are no other branches and no forks.

## THE CODE IS NOT READABLE YET — and that is Lonnie's call, not mine

The Director can read the relay right now and cannot read a line of the code. Opening it is a
decision I do not get to make. There are two ways in, and Lonnie picks one or neither:

1. **Add the Director as a read collaborator.** Settings → Collaborators → add, role *Read*.
   Nothing becomes public.
2. **Make the repository public.** Settings → General → Change visibility.

**What I checked before saying that, so the choice is made on facts:** the committed tree carries
**no credentials**. I scanned every tracked file for API keys and found none. The funded anchor key
lives outside the repository by design and always has, and the live `sk-proj-` key Lonnie has ruled
acceptable for his own team is in a directory that is not part of this repository. So publishing
would not leak a key. **It would still publish the whole design**, which is the part that is his to
weigh and not mine.

Until one of those happens, everything below is a map the Director cannot yet open.

## What is at HEAD

```
dda9e7a  210.E: the bench panel — give it a soul, swap it, lock it, and see what it holds
f3d4220  210.E: the soul is given to the mind that actually runs
322be2c  210: the suite — 33 checks
f367a6c  210.D: the soul is fed into the wires that existed
ed959c0  210: the persona system
```

Every directive is one commit or a short run of them, and the commit message names the directive
number. `git log --oneline --grep='^210'` finds a directive's whole build.

## Where the mind lives

Everything is under `server/src/`.

| file | what it is |
|---|---|
| `persona.js` | **the soul** — 210. Rolled, derived, authored; the Genesis lock; the cited mapping table |
| `interests.js` | what it has come to care about — grown threads, and the struck layer the soul supplies |
| `offers.js` | the offers model — what an act offers, what it is worth, and the soul's pull |
| `goals.js` | the choosing |
| `safety.js` | the gate everything runs behind — 203 |
| `curiosity.js` | novelty and coping potential — 201/204 |
| `sleep.js` | the life clock, tiredness, dreaming, the waking wonder |
| `dictionary.js` | the only place meaning resolves — 191 |
| `glyphs.js` | Lonnie's 402 marks, indexed out of his own sheets |
| `watching.js` | **the Mind Emulator's mind** — the tick, and the only place the whole thing runs |
| `demo-watching.js` | the Emulator's page — what he watches it on |
| `accept.js` | runs every proving suite |
| `acceptance-*.js` | one suite per directive; each check names the clause it proves |

## Running it

```
cd CC-Wanderer && npm run accept          # every suite, from the repo root, not from server/
cd CC-Wanderer/server && node src/demo-watching.js   # the Mind Emulator, 127.0.0.1 only
```

The Emulator needs the local model service (`ollama serve`) up first, and it binds to this machine
only — there is no remote way in, by design.

## The one thing worth knowing about the checks

**A suite passing is not a verdict.** It says the code does what the directive asked for in words.
Whether the mind is any good is Lonnie's eye on the Emulator, and nothing in this repository is
allowed to claim otherwise.
