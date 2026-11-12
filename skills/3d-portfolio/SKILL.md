---
name: 3d-portfolio
description: Build a personal portfolio site with a 3D hero, animated objects for each interest, visual case studies and a working contact form (tRPC, Resend, Turnstile, Postgres), then deploy it to Vercel. Use when someone asks for a portfolio, personal site or "3D portfolio" and wants something more distinctive than a template.
---

# 3D portfolio

Build a one-page portfolio that could only belong to this person. The site that
produced this skill was built around a DJ's record crate: every project is a
record sleeve you flick through, and the record spins when it is in front.
Example: https://alessio-simoya.vercel.app

Do not copy that concept. Follow the process and find theirs.

## 1. Collect the inputs first

Ask for these before designing. Build with real content from the start.

- **Projects:** for each one, a short brief of facts: what it is, their role,
  dates, what it did, the stack, a live link, and a list of things that must
  not be claimed.
- **Experience:** company, title, dates, place, and one result per job.
- **Skills:** grouped, with where each was proven.
- **Life outside work:** hobbies, teams, what they cook, where they have been.
  This is where the concept comes from, so ask for more than they offer.
- **Assets:** logos, product screenshots, videos, Rive or Lottie files. Ask
  where the project folders live and search them.
- **Links:** LinkedIn, GitHub and so on. Never publish their email address
  unless they say to.

## 2. Find the one concept

Pick the most characteristic thing in the person's world and make it the hero,
as a real 3D object people can play with. A DJ gets a crate of records. A
climber might get a wall of holds; a baker, a tray of loaves. Each project
becomes one item in that object.

Spend the boldness there. Keep the rest of the page quiet: flat colour fields,
one display typeface with character, one plain body face, four or five named
colours.

## 3. Stack

- Vite, React and TypeScript in strict mode
- three.js for 3D, written directly (no scene framework needed)
- Rive only if the person already has `.riv` files; lazy-load its wasm
- tRPC and zod for the API, as a Vercel function at `api/trpc/[trpc].ts`
- Resend for email, Cloudflare Turnstile for spam, Neon Postgres for records
- Self-hosted variable fonts through `@fontsource-variable/*`

## 4. How the 3D is put together

Use **one** fixed, full-screen WebGL canvas with `pointer-events: none`, and
draw every 3D view into the rectangle of a DOM placeholder:

```ts
const r = el.getBoundingClientRect();
renderer.setViewport(r.left, innerHeight - r.bottom, r.width, r.height);
renderer.setScissor(r.left, innerHeight - r.bottom, r.width, r.height);
renderer.render(scene, camera);
```

Clear once per frame, turn the scissor test on, and skip any element that is
off screen. This gives many 3D objects down the page for the cost of one
context, and they scroll with the layout.

Give each small object a "rig": a group, a camera position, a look-at point and
a `tick(t)` function driven by one shared clock. Build rigs lazily and show one
at a time. Draw textures on a 2D canvas (logos, labels, patterns) so there are
no model files to load.

Things that went wrong and how to avoid them:

- A ground-shadow plane larger than the element shows as a hard rectangle.
  Size it to fit inside the view.
- Rotate a group, then spin a child, when you need "spin about its own axis".
- Sprites and transparent planes draw after solid meshes; a glow under an
  object should be a flat plane below it, not a camera-facing sprite.

## 5. Sections that worked

1. **Hero:** name set very large, a sticker that cycles through their roles,
   one sentence, and the interactive object. A coloured disc behind it changes
   with the item in focus.
2. **Work:** each case study opens on a band in that brand's colour with its
   real logo, then a sideways reel of images, video and animation, three short
   highlights, a few large figures and the stack as chips. Long text goes
   behind "Read the full story".
3. **Experience:** one row per job with one large number each.
4. **Journey:** schools or milestones, each with its own animated object.
5. **Skills:** rows of moving type, not a table.
6. **Off the clock:** one animated object per interest.
7. **Travels:** places as fridge magnets on a steel door that can be dragged.
8. **Say hello:** the contact form.

A floating pill header marks the current section with an
`IntersectionObserver`.

## 6. House rules

- **Facts only.** Use what the person supplied. If a section needs something
  they did not give, leave it out. Keep each project's "do not claim" list in
  the content file as a comment.
- **No vanity counts.** Commits, lines of code, number of pages or models read
  as immature. Show the product and what it did.
- **Show less text.** If a screen is mostly paragraphs, turn it into a visual.
- **Respect other people's work.** Do not rebuild a paid template from its
  preview, and do not copy a brand's product design (a named match ball, for
  example); draw an original in the same spirit. Use only images that are
  freely licensed, and credit them on the page.
- **Screenshots** must not show real customers' names, amounts or faces.
- **Motion has an off switch.** Honour `prefers-reduced-motion`, add a "Pause
  motion" button, and give a flat fallback when WebGL is missing.
- **Put all content in one typed file** (`src/content/portfolio.ts`) so the
  person can edit words without touching components.

## 7. The contact form

One tRPC procedure, `contact.send`, with zod input. In order, it:

1. returns success silently if the hidden honeypot field is filled;
2. applies a small per-address rate limit;
3. verifies the Turnstile token with Cloudflare, and refuses if it fails;
4. emails the owner through Resend, with the sender as `reply_to`;
5. saves every message to a `messages` table;
6. only if the visitor ticked an unticked-by-default "keep me posted" box,
   saves them to a `subscribers` table (with the consent wording and time) and
   to a Resend audience.

Never add people to a mailing list without that tick.

Environment variables: `RESEND_API_KEY`, `ENQUIRY_TO_EMAIL`,
`ENQUIRY_FROM_EMAIL`, `RESEND_AUDIENCE_ID`, `DATABASE_URL`,
`TURNSTILE_SECRET`, and the public site key for the browser.

Things that went wrong:

- The `from` value may already be `Name <address>`. Wrapping it again makes
  Resend answer 422. Use it as given when it contains `<`.
- Vite only exposes variables that start with a prefix in `envPrefix`. If the
  site key arrives under another name, add that prefix.
- A Turnstile widget only works on hostnames listed for it in Cloudflare.
- In production, with no Turnstile secret, refuse to send. Use Cloudflare's
  published test keys only in local development.
- Add a `CONTACT_DRY_RUN=1` switch, ignored in production, so the whole path
  can be tested without sending email.
- To run the same handler under `vite dev`, add a small plugin that loads the
  function with `server.ssrLoadModule` and passes it a `Request`.
- Build email HTML with tables and inline styles, and escape every field.

## 8. Check your work as you go

After each round of changes:

- run the build;
- take Playwright screenshots of every section at 1440 and 390 pixels wide
  (launch Chromium with `--use-gl=swiftshader` so WebGL renders headless);
- confirm `document.documentElement.scrollWidth - innerWidth` is 0;
- look at the screenshots yourself before saying it is done.

Say plainly what was not checked: real phones, Safari, the feel of the motion,
real email delivery.

## 9. Ship it

- `git init`, with `.env*` and `.vercel` ignored. Scan staged files for keys
  before the first commit.
- `vercel link`, add the environment variables, `vercel --prod`.
- Check the live page loads without failed requests and that the API rejects a
  request with no Turnstile token.
- Ask the owner to send one real message through the live form.

## Starter prompt

Give the person this to fill in:

```
Build my 3D portfolio using the 3d-portfolio skill.

Name and one-line description of me:
Roles to cycle in the hero (4 to 6):
Projects (for each: facts, my role, dates, link, what not to claim):
Experience (company, title, dates, one result each):
Skills, grouped:
Schools or milestones:
Interests outside work (be specific):
Places I have been:
Where my logos, screenshots and animations live on this machine:
Links to show (LinkedIn, GitHub and so on):
Things I never want on the site:
```
