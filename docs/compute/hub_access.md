# Hub Access Overview

There are two ways to access the JupyterHub depending on your workflow preferences. You can:

- Access the Hub directly through a *web browser*
- Connect to the Hub using *VSCode on your local machine*

This page can help you decide which method is right for you based on your workflow.

| Method                   | Best For                                                 |
| ------------------------ | -------------------------------------------------------- |
| **Browser (JupyterLab)** | New users, teaching, notebooks, simple experiments       |
| **Local VS Code IDE**    | Power users who want full desktop performance & features |

## Method Comparison

### 1. **Browser (JupyterLab / Web UI)**

**Pros**

- Fastest onboarding, works from any machine
- Zero local setup or install
- Guaranteed environment parity with Hub compute
- Ideal for teaching, workshops, quick data exploration

**Cons**

- Not a full IDE (limited refactoring, UX, extensions)
- Multi-file repo workflows can feel clunky
- Fewer features compared to VS Code

**Use-cases**

- First-time users, classes, tutorials
- Notebook-centric workflows
- Reliable & consistent shared environments

See installation instructions here: [Access via Web Browser](hub_browser_access.md)

### 2. **Local VS Code IDE Connected to the Hub**

**Pros**

- Richest developer experience (UI, extensions, keybindings)
- Ideal for long sessions and power-user workflows
- Full VS Code features, debugging, linting, Git integrations

**Cons**

- Setup can be fragile (SSH, token config, port-forwarding)
- Easy to mistakenly use your **local** interpreter/kernel
- Credentials or data may unintentionally end up stored locally
- More complex to support in classrooms/workshops

**Use-cases**

- Power users comfortable with remote dev tools
- Working on large codebases or long-running services
- Custom local workflows that the Hub can’t support directly

See installation instructions here: [Access via VSCode (external)](vs_code_to_hub.md)

## Installing Packages & Environment Behavior

| Behavior                               | Browser                                   | Local VS Code IDE                                     |
| -------------------------------------- | ----------------------------------------- | ----------------------------------------------------- |
| **Where packages install**             | On the Hub compute                        | Depends on interpreter—can be local or remote         |
| **Persistence across server restarts** | Yes, if installed in `$HOME` or Conda env | Same, if using remote interpreter                     |
| **Common pitfall**                     | None                                      | Accidentally installing locally instead of on the Hub |

!!! note

    Go to [Managing Software](manage_software.md) for more guidance on installing packages.

## Security & Data Movement

- *Browser*: data stays on the Hub unless explicitly downloaded
- *Local VS Code IDE*: easier to accidentally sync data to your local machine or expose secrets via cached credentials

!!! tip

    You can switch between methods at any time - no need to stick to one - All methods access the same files in your Hub home directory
