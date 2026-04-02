# cesau-skills

Shared Cursor skills for use across projects as a git submodule.

## Adding to a project

```bash
git submodule add <repo-url> .cursor/skills
git commit -m "add cesau-skills submodule"
```

## Updating skills in a project

```bash
git submodule update --remote .cursor/skills
git commit -m "update cesau-skills"
```

## Cloning a project that uses this

```bash
git clone --recurse-submodules <project-url>
```

Or if already cloned:

```bash
git submodule update --init
```
