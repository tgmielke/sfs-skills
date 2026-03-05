# sfs-skills

Claude and Cursor skills for Salesforce Field Service configuration.

## Available Skills

### sf-fs-datacapture

Creates Field Service **data capture form flows** from images, PDFs, or descriptions of existing paper/digital forms. Generates mobile-optimized `DataCaptureFlow` XML with:

- **Field Service DC components** (`dcTextInput`, `dcPicklist`, `dcToggle`, `dcCbGroup`, `dcUpImage`, `dcSignature`)
- **Voice-to-form** support (`IsLlmTargetable`) for hands-free data entry
- **Offline** environment for field technician mobile use
- **Conditional visibility** with toggle-based show/hide patterns
- **Photo capture** and **signature** components

### Workflow

```
Image/PDF of form → Analyze fields → Generate DataCaptureFlow XML → Deploy to org
```

Share a photo or PDF of any paper form, inspection checklist, or data capture document, and the agent will produce a deployable `.flow-meta.xml` file.

### Skill Structure

```
skills/sf-fs-datacapture/
├── SKILL.md                                          # Main skill instructions
├── assets/
│   └── data-capture-flow-template.xml                # Base XML template
└── references/
    ├── example-safety-hazard-check.xml               # Real flow example
    ├── example-customer-interaction.xml              # Real flow example
    ├── example-nas-demarc-validation.xml             # Real flow example
    ├── field-type-mapping.md                         # DC component patterns
    └── mobile-ux-guide.md                            # Mobile UX best practices
```

## Installation

### Cursor

Copy the skill directory to your personal skills folder:

```bash
cp -r skills/sf-fs-datacapture ~/.cursor/skills/
```

### Claude Code

Copy the skill directory to your Claude Code skills folder:

```bash
cp -r skills/sf-fs-datacapture ~/.claude/skills/
```

## Usage

Once installed, share an image or describe a form and ask the agent to create a data capture flow:

- *"Here's a photo of our safety inspection form — create a data capture flow from it"*
- *"Build a data capture form for equipment installation with serial number, model, condition check, and technician sign-off"*
- *"Convert this PDF checklist into a Field Service data capture flow and deploy it to my demo org"*

## Prerequisites

- Salesforce CLI v2 (`sf` command)
- Authenticated Salesforce org with Field Service enabled
- `sfdx-project.json` in your project root (for deployment)

## License

MIT
