# 3d-portfolio skill

A Claude Code skill for building a personal portfolio with a 3D hero, animated
objects, visual case studies and a working contact form, then deploying it to
Vercel.

It was written from building https://alessio-simoya.vercel.app and records the
process, the architecture and the mistakes worth avoiding.

## Install

Copy the skill folder into your Claude Code skills directory:

```
git clone https://github.com/walenso/3d-portfolio-skill
mkdir -p ~/.claude/skills
cp -r 3d-portfolio-skill/skills/3d-portfolio ~/.claude/skills/
```

## Use

Start Claude Code in an empty folder and say:

```
Build my 3D portfolio using the 3d-portfolio skill.
```

The skill ends with a starter prompt listing what to have ready: your
projects, experience, skills, interests and where your assets live.
