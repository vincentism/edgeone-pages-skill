---
name: edgeone-pages-deploy
description: Deploy projects to EdgeOne Pages platform. Covers CLI installation, authentication (browser login and token-based), site selection (China/Global), and deployment. Load this skill when a user wants to deploy a frontend or full-stack project to EdgeOne Pages.
metadata:
  author: edgeone
  version: "1.0.0"
---

# EdgeOne Pages Deployment Skill

This skill guides how to deploy projects to the **EdgeOne Pages** platform. It applies to static frontend projects as well as full-stack projects using Edge Functions or Node Functions.

---

## Deployment Workflow Overview

```
Step 1: Detect and install CLI
Step 2: Check login status
Step 3: Login (site selection + browser login / token login)
Step 4: Deploy
Step 5: Output results
```

---

## Step 1: Detect and Install EdgeOne CLI

> **⚠️ Version Requirement**: This skill currently requires **edgeone@1.2.9-beta.11** specifically. If the CLI is already installed but at a different version, it must be reinstalled at the specified version.

### Check if installed

```bash
edgeone -v
```

- If the output shows `1.2.9-beta.11` → CLI is ready, proceed to Step 2.
- If the CLI is not found or the version is different → install (or reinstall) the specified version:

### Install the required version

```bash
npm install -g edgeone@1.2.9-beta.11
```

After installation, run `edgeone -v` again to confirm it shows `1.2.9-beta.11`.

---

## Step 2: Check Login Status

> **Note**: If the user explicitly chooses to deploy with a Token (via the `-t` flag), skip Step 2 and Step 3 entirely — token-based deployment does not require prior login. Go directly to **Step 4 (Token scenario)**.

```bash
edgeone whoami
```

- If user info is returned → already logged in, skip to **Step 4**.
- If an error or "not logged in" message appears → proceed to **Step 3**.

---

## Step 3: Login

EdgeOne Pages has two sites: **China** and **Global**. The site must be determined before login.

### Step 3a: Determine Site

**You must ask the user to choose a site before running the login command.**

Use the IDE's selection control (`ask_followup_question`) to prompt the user:

> Choose your EdgeOne Pages site:
> - **China** — For users in mainland China. Console: console.cloud.tencent.com
> - **Global** — For users outside China. Console: console.intl.cloud.tencent.com

If the selection control is unavailable (e.g., in a CLI environment), ask in plain text:
> "Would you like to deploy to EdgeOne Pages China or Global site?"

### Step 3b: Execute Login

**The agent must automatically detect the environment and choose the appropriate login method.** Follow this decision logic:

#### Environment Detection

Run the following command to check if the current environment supports opening a browser:

```bash
# macOS
which open 2>/dev/null && echo "BROWSER_AVAILABLE" || echo "NO_BROWSER"

# Linux
which xdg-open 2>/dev/null && echo "BROWSER_AVAILABLE" || echo "NO_BROWSER"
```

**Decision rules:**
1. If the output contains `BROWSER_AVAILABLE` → use **Browser Login** (Method 1)
2. If the output is `NO_BROWSER`, or the environment is a remote server / SSH session / CI/CD pipeline → use **Token Login** (Method 2)
3. If the user has explicitly requested token-based deployment (e.g., via `-t` flag) → use **Token Login** regardless of environment

#### Method 1: Browser Login (default when browser is available)

Run the corresponding command based on the selected site:

```bash
# China site
edgeone login --site china

# Global site
edgeone login --site global
```

This will automatically open a browser for the user to complete Tencent Cloud account login. The CLI will receive the login credentials automatically.

**Note**: Wait for the user to complete login in the browser. The CLI will display a success message. If there is no response for an extended period, prompt the user to check whether the browser has opened the login page.

#### Method 2: Token Login (fallback for headless environments or CI/CD)

Use token-based login when browser login is unavailable:
- Remote server / SSH terminal (no browser available)
- CI/CD pipeline
- Any environment where browser login is impractical
- User explicitly chooses token-based deployment

**Token login does NOT require running `edgeone login`**. Instead, the token is passed directly via the `-t` flag in the deploy command.

Guide the user to obtain an API Token:

1. Direct the user to the console page based on the selected site:
   - **China**: https://console.cloud.tencent.com/edgeone/pages?tab=settings
   - **Global**: https://console.intl.cloud.tencent.com/edgeone/pages?tab=settings
2. Locate the **API Token** settings section
3. Click **Create Token**
4. Copy the generated token

**Security warning**: Remind the user that the token carries account-level permissions. It should never be leaked or committed to a code repository. Once obtained, the token is used directly in the deploy command (see Step 4).

---

## Step 4: Deploy

### Scenario 1: Already logged in via browser

Check whether an `edgeone.json` file exists in the project root:

- **`edgeone.json` exists** (project already linked):

```bash
edgeone pages deploy
```

- **`edgeone.json` does not exist** (new project):

```bash
edgeone pages deploy -n <project-name>
```

`<project-name>` should be auto-generated by the agent based on the project name or directory name. If the project does not exist on the platform, it will be created automatically.

> **Note**: After a successful first deployment with `-n`, the CLI automatically generates an `edgeone.json` file in the project root. Subsequent deployments will use this file and no longer require the `-n` flag.

### Scenario 2: Token-based deployment

Token deployment does not require running `edgeone login` beforehand. Pass the token directly in the deploy command:

```bash
# Project already has edgeone.json
edgeone pages deploy -t <token>

# New project
edgeone pages deploy -n <project-name> -t <token>
```

> **Note**: The token is already bound to a specific site (China or Global), so there is no need to specify `--site` when deploying with a token.

### Deployment Environment

Deployments target the **production** environment by default. To deploy to a preview environment:

```bash
edgeone pages deploy -e preview
```

### Automatic Build Behavior

- The CLI automatically detects the project framework and runs the build
- The CLI automatically determines and uploads the build output directory (e.g., `dist`, `out`, `build`, etc.)
- No manual specification of the build output path is needed

---

## Step 5: Deployment Results

After a successful deployment, the CLI outputs several key pieces of information. **You must extract the full value of `EDGEONE_DEPLOY_URL` as the access URL.**

### Output Format Example

```
[cli][✔] Deploy Success
EDGEONE_DEPLOY_URL=https://my-project-abc123.edgeone.cool?eo_token=xxxx&eo_time=yyyy
EDGEONE_DEPLOY_TYPE=preset
EDGEONE_PROJECT_ID=pages-xxxxxxxx
[cli][✔] You can view your deployment in the EdgeOne Pages Console at:
https://console.cloud.tencent.com/edgeone/pages/project/pages-xxxxxxxx/deployment/xxxxxxx
```

### Extraction Rules (⚠️ Critical)

1. **Access URL**: Find the line starting with `EDGEONE_DEPLOY_URL=` and extract the **complete URL** after `=`, including the `eo_token` and `eo_time` query parameters. **Do NOT truncate or omit these parameters** — without them, the user will be unable to access the page.
2. **Project ID**: Extract from the `EDGEONE_PROJECT_ID=` line.
3. **Console URL**: Extract from the line following `You can view your deployment in the EdgeOne Pages Console at:`.

### Display to User

Present the extracted information to the user. **The access URL must include the full `eo_token` and `eo_time` parameters**:

> ✅ Deployment complete!
> - **Access URL**: `https://my-project-abc123.edgeone.cool?eo_token=xxxx&eo_time=yyyy`
> - **Console URL**: `https://console.cloud.tencent.com/edgeone/pages/project/...`

### Error Handling

If the deployment fails:

1. **Build failure**: Check the build logs in the CLI output. Common causes include missing dependencies (`npm install` not run) or misconfigured build scripts in `package.json`.
2. **Authentication failure**: Run `edgeone whoami` to verify login status. Re-login if needed, or verify the token is valid and not expired.
3. **Network timeout**: Retry the deployment. If the issue persists, check network connectivity.
4. **Project name conflict**: If the project name is already taken by another account, choose a different name with the `-n` flag.

---

## Appendix: Edge Functions / Node Functions Projects

If the project requires **Edge Functions** or **Node Functions** (server-side function capabilities), run initialization before deployment:

```bash
edgeone pages init
```

This command generates an `edge-functions` or `node-functions` folder with sample functions based on the guided prompts.

**Pure static frontend projects do not need `init` and can be deployed directly.**

---

## Appendix: Local Development

To debug locally before deployment:

```bash
edgeone pages dev
```

- Runs on `http://localhost:8088/` by default
- Frontend pages and function services share the same port

---

## Appendix: Environment Variable Management

```bash
# List all environment variables
edgeone pages env ls

# Pull environment variables to a local .env file
edgeone pages env pull

# Add an environment variable
edgeone pages env add KEY value

# Remove an environment variable
edgeone pages env rm KEY
```

---

## Appendix: Project Linking

To use KV storage or sync console environment variables for local development, link the project first:

```bash
edgeone pages link
```

Follow the prompts to enter an existing Pages project name. If the project does not exist, the CLI will guide you to create a new one.

---

## FAQ

| Problem | Solution |
|---------|----------|
| `command not found: edgeone` | Run `npm install -g edgeone@1.2.9-beta.11` to install the CLI |
| Browser does not open during login | Check default browser settings, or use token-based login instead |
| Deployment fails with "not logged in" | Run `edgeone whoami` to check login status and re-login |
| Unsure whether to choose China or Global | Mainland China users → China; users outside China → Global |
| Where to get an API Token | China: console.cloud.tencent.com/edgeone/pages?tab=settings; Global: console.intl.cloud.tencent.com/edgeone/pages?tab=settings |
| Token deployment fails with auth error | Verify the token is correct and has not expired; regenerate if needed |

---

## Command Reference

| Action | Command |
|--------|---------|
| Install CLI | `npm install -g edgeone@1.2.9-beta.11` |
| Check version | `edgeone -v` |
| Show help | `edgeone -h` |
| Login (China) | `edgeone login --site china` |
| Login (Global) | `edgeone login --site global` |
| View login info | `edgeone whoami` |
| Logout | `edgeone logout` |
| Switch account | `edgeone switch` |
| Initialize functions | `edgeone pages init` |
| Local development | `edgeone pages dev` |
| Link project | `edgeone pages link` |
| Deploy | `edgeone pages deploy` |
| Deploy to new project | `edgeone pages deploy -n <name>` |
| Deploy to preview env | `edgeone pages deploy -e preview` |
| Deploy with token | `edgeone pages deploy -t <token>` |
| List env variables | `edgeone pages env ls` |
| Pull env variables | `edgeone pages env pull` |
