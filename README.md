# agent-skills

This is your personal AI agent skills library powered by [Hanka](https://usehanka.com).

## What is this?

Store and share reusable AI agent skills that can be installed by any AI agent or developer.

## Installing Skills

### For Public Repos

```bash
npx skills add zaaakher/agent-skills --skill skillname
# or add all skills from the repo
npx skills add zaaakher/agent-skills
```

### For Private Repos

First, clone the repository locally:

```bash
git clone git@github.com:zaaakher/agent-skills.git ~/temp-skills
```

Then add skills from the local clone:

```bash
cd your-project
npx skills add ~/temp-skills --skill "Your Skill Name"
# or add all skills
npx skills add ~/temp-skills
```

## Adding Skills

1. Go to [usehanka.com](https://usehanka.com) and sign in
2. Select this repository as your skill library
3. Create your first skill using the dashboard

## Structure

- `.hanka/index.json` - Lists all available skills in this repository
- `.hanka/config.json` - Configuration for your skill library
- `*.md` - Skill definition files in Markdown format

## Making Skills Public

By default, skills are private. To share a skill publicly, edit the skill and toggle "Public" on. Public repos can be discovered and installed by anyone.

## Learn More

- [Hanka Documentation](https://docs.usehanka.com)
- [GitHub](https://github.com/usehanka/hanka)
