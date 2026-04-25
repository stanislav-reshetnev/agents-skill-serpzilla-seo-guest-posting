---
name: serpzilla-seo-guest-posting
description: Skill for SEO promotion by purchasing guest posts on trusted donor websites using the Serpzilla platform
version: 1.0.0
author: Stanislavus
user-invocable: true
triggers:
  - buy guest post for SEO
  - find donor site for guest posting
  - serpzilla search donors
  - configure serpzilla mcp
metadata:
  {
    "openclaw":
      {
        "emoji": "🦖",
        "requires": { "bins": ["mcporter"] },
      },
  }
---

# Serpzilla SEO Guest Posting

## Description

This skill is designed for SEO specialists and webmasters who want to improve their website's backlink profile.
It allows you to automate the search and purchase of guest posts (article placements on donor sites 
that pass link equity to your promoted resource) through OpenClaw. Interaction with the platform is done 
via the Serpzilla MCP server, managed by the `mcporter` tool.

## Prerequisites

The following components must be installed and configured on the server where OpenClaw runs:

1. **Docker** – to run the Serpzilla MCP server in an isolated container.
2. **Node.js and npm** – to install and run `mcporter`.
3. **OpenClaw** – the agent runtime environment.

Before using any Serpzilla commands, verify the MCP server is already registered with `npx mcporter list serpzilla`.

## Preliminary Setup

To fully use this skill, you need to register and configure the connection to the Serpzilla API.

### 1. Registration and API token

1. Register as an advertiser on [Serpzilla.com](https://serpzilla.com/).
2. Go to the security section at [https://passport.serpzilla.com/security/token/](https://passport.serpzilla.com/security/token/).
3. Copy your **API token** and remember your **login** (email) used for registration.

### 2. Install mcporter and add the Serpzilla MCP server

`mcporter` is a CLI tool for managing MCP servers. OpenClaw uses it to communicate with external services.

**Install mcporter globally:**
```bash
npm install -g mcporter
```

Add the Serpzilla MCP server:

It is necessary to request the user to provide values for the environment variables SERPZILLA_LOGIN
(the user's chosen login for registering with serpzilla.com, usually an email address)
and SERPZILLA_API_TOKEN (the API token received by the user).
Run the following command, replacing XXX and YYY with user's values:

```bash
npx mcporter config add serpzilla --stdio "sudo docker run -i --rm --env SERPZILLA_LOGIN=XXX --env SERPZILLA_API_TOKEN=YYY serpzilla-mcp-stdio-server"
```

### 3. Verify the configuration

Check that the MCP server was added successfully:

```bash
npx mcporter list serpzilla
```

## Using the skill

After completing the preliminary setup, the skill is ready to use. The typical workflow for purchasing
a guest post is described below.

### Step 1: Create a project for your promoted website

Create a project inside Serpzilla for your domain. Replace **YOUR_SITE.COM** with your actual website URL:

```bash
npx mcporter call serpzilla.create_project domain=YOUR_SITE.COM
```

*Important:* Save the returned **PROJECT_ID** – you will need it later.

To view all your existing projects:

```bash
npx mcporter call serpzilla.list_projects
```

### Step 2: Search for donor sites

With your **PROJECT_ID**, you can search for suitable donor sites. Basic search command:

```bash
npx mcporter call serpzilla.search_sites project_id=PROJECT_ID link_type=link
```

To see all available filtering parameters:

```bash
npx mcporter list serpzilla --schema --json
```
