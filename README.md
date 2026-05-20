# MidCamp Event Platform

MidCamp 2026 and onward! This project is built on the [Event Platform](https://www.drupal.org/project/event_platform) distribution with the [Event Horizon](https://www.drupal.org/project/event_horizon) theme.

## Table of Contents

- [Requirements](#requirements)
- [Local Development Setup](#local-development-setup)
  - [Prerequisites](#prerequisites)
  - [Getting Started](#getting-started)
- [Common Development Tasks](#common-development-tasks)
  - [Running Drush Commands](#running-drush-commands)
  - [Database Management](#database-management)
  - [Cache Management](#cache-management)
  - [Configuration Management](#configuration-management)
  - [Accessing the Site](#accessing-the-site)
- [Remote Environment](#remote-environment)
  - [Lagoon Deployment](#lagoon-deployment)
  - [Environment Types](#environment-types)
  - [Automated Tasks](#automated-tasks)
- [Event Platform Starter](#event-platform-starter)
  - [About the Starter Recipe](#about-the-starter-recipe)
  - [Initial Configuration](#initial-configuration)
  - [Optional Features](#optional-features)
- [Theme Development](#theme-development)
- [Troubleshooting](#troubleshooting)
- [Project Information](#project-information)

---

## Requirements

- **DDEV**: Local development environment

---

## Local Development Setup

### Prerequisites

Install [DDEV](https://ddev.readthedocs.io/en/stable/) for your operating system.

Set up an account and ssh key for [Amazee.io Lagoon](https://dashboard.amazeeio.cloud/projects/midcamp-ep), as this project deploys there.

### Getting Started

1. **Clone the repository**
   ```bash
   git clone https://github.com/MidCamp/midcamp-ep.git
   cd midcamp-ep
   ```

2. **Start DDEV**
   ```bash
   ddev start
   ```

3. **Install Composer dependencies** Restarting DDEV ensures that `settings.php` gets updated with DDEV includes.
   ```bash
   ddev composer install
   ddev restart
   ```

4. **Install the site from the Lagoon remote**

   ```bash
   ddev auth ssh
   ddev pull lagoon
   ddev drush cr
   ```

5. **Access your site**
   ```bash
   ddev launch
   ```

   Default URL: `https://midcamp-ep.ddev.site`

---

## Common Development Tasks

### Syncing environment from prod to dev

```bash
ddev pull lagoon --skip-files
ddev push lagoon --environment=LAGOON_ENVIRONMENT=dev --skip-files
```

### Accessing the remote

```bash
ddev drush @lagoon.midcamp-ep-main ssh
```

### Running Drush Commands

```bash
ddev drush <command>
```

### Database Management

**Export database:**
```bash
ddev export-db --file=backup.sql.gz
```

**Import database:**
```bash
ddev import-db --file=backup.sql.gz
```

**Create database snapshot:**
```bash
ddev snapshot
ddev snapshot restore
```

### Cache Management

**Clear all caches:**
```bash
ddev drush cr
```

### Configuration Management

**Export configuration:**
```bash
ddev drush config:export
# or
ddev drush cex
```

**Import configuration:**
```bash
ddev drush config:import
# or
ddev drush cim
```

### Accessing the Site

**Get login link:**
```bash
ddev drush uli
```

**Access database:**
```bash
ddev mysql
```

---

## Remote Environment

This project is configured to deploy to [Amazee.io Lagoon](https://www.amazee.io/), an enterprise hosting platform for Drupal.

### Lagoon Deployment

The project is configured in `.lagoon.yml` with the following settings:

- **Project Name**: `midcamp-ep`
- **Main Branch**: `main` (receives automated deployments)

**Services:**
- **CLI**: Command-line interface with persistent storage
- **NGINX**: Web server with persistent file storage
- **PHP-FPM**: PHP processor
- **MariaDB 10.11**: Database server

### Environment Types

Lagoon supports multiple environment types:
- **Production**: The main branch deployment
- **Development**: Feature branch deployments

To deploy a feature branch, push it to your Git remote. Lagoon will automatically create an environment.

### Automated Tasks

**Post-Rollout (after deployment):**
1. Database updates: `drush updb`
2. Cache rebuild: `drush cr`

**Cron Jobs (main environment):**
- Hourly Drupal cron: `drush cron`

**Pre-Rollout (commented out by default):**
- Database backup option available - uncomment in `.lagoon.yml` if needed

### Interacting with Remote

**SSH into environment:**
```bash
ssh -t <environment>@ssh.lagoon.amazeeio.cloud
```

**Run Drush commands remotely:**
```bash
ssh <environment>@ssh.lagoon.amazeeio.cloud drush <command>
```

**View logs:**
```bash
lagoon logs -p midcamp-ep -e <environment>
```

Consult your Lagoon administrator or documentation for specific connection details and credentials.

---

## Event Platform Starter

### About the Starter Recipe

The Event Platform Starter recipe (`recipes/event_platform_starter/`) provides a quick way to bootstrap your event website with:

- **Event Platform**: Core event management functionality
- **Event Horizon Theme**: Companion theme optimized for event sites
- **Drupal CMS Recipes**: Additional functionality for a robust site
  - Admin UI improvements
  - Anti-spam protection
  - SEO basics
  - Responsive images

### Initial Configuration

The Event Platform Starter recipe cannot be applied directly via `drush recipe` due to dependency optimization issues. Instead, follow these manual steps:

1. **Enable Event Platform and dependencies:**
   ```bash
   ddev drush en event_platform -y
   ddev drush en moderation_state_condition -y
   ```

2. **Install and set Event Horizon theme:**
   ```bash
   ddev drush thin event_horizon
   ddev drush config:set system.theme default event_horizon -y
   ```

3. **Apply example configuration** (includes a first event):
   ```bash
   ddev drush recipe ../recipes/event_platform_example
   ```

4. **Install recommended modules for better admin experience:**
   ```bash
   ddev drush en keysave -y
   ddev drush en navigation_extra_tools -y
   ```

5. **Apply Drupal CMS recipes:**
   ```bash
   ddev drush recipe ../recipes/drupal_cms_admin_ui
   ddev drush recipe ../recipes/drupal_cms_anti_spam
   ddev drush recipe ../recipes/drupal_cms_seo_basic
   ddev drush recipe ../recipes/drupal_cms_image
   ```

   **Note:** Some recipes may fail with configuration exists errors. You can safely ignore these for now and proceed.

### Optional Features

**Personal Schedules (Flagging):**

Enable the ability for users to flag sessions and create personal schedules:
```bash
ddev drush en event_platform_flag -y
```

**Responsive Images:**

After applying the `drupal_cms_image` recipe, update display formatters on your content types to use the new responsive image styles.

### Configuration Sources

This project includes two Event Platform recipes in the `recipes/` directory:

- `event_platform_starter/`: Base setup instructions and dependencies
- `event_platform_example/`: Example configuration with sample event content

Both recipes are sourced from the contrib Event Platform module and provide guidance for initial setup.

---

## Theme Development

The custom theme is located at: `web/themes/custom/midcamp_event_horizon/`

This theme is based on Event Horizon and includes:

**Live Reload (Development):**
```bash
cd web/themes/custom/midcamp_event_horizon
npm install
npm run livereload
```

The livereload script watches for changes in CSS, JS, Twig templates, and image files.

**Theme Structure:**
- Templates: Standard Drupal theme structure
- Assets: Theme-specific styles and scripts
- Bootstrap 4: Framework for styling (as per project defaults)

---

## Troubleshooting

### DDEV Issues

**Site not accessible:**
```bash
ddev restart
ddev describe
```

**Container conflicts:**
```bash
ddev stop --all
ddev start
```

**Clear DDEV cache:**
```bash
ddev clean
ddev restart
```

### General Drupal Issues

**Module or configuration issues:**
```bash
ddev drush cr
ddev drush updb -y
ddev drush cim -y  # If config sync is enabled
```

**Database connection issues:**

Check `web/sites/default/settings.php` and ensure database credentials match your environment.

**File permission issues:**
```bash
chmod -R 755 web/sites/default/files
```

---

## Project Information

- **Homepage**: [https://www.midcamp.org](https://www.midcamp.org)
- **License**: GPL-2.0-or-later
- **Drupal Version**: 11.2
- **PHP Version**: 8.3

### Key Dependencies

- **Event Platform**: Event management distribution
- **Event Horizon**: Theme for event websites
- **Drupal CMS**: Enhanced admin experience modules
- **Additional Modules**:
  - Coffee (admin menu search)
  - Gin (admin theme)
  - Pathauto (automatic URL aliases)
  - Honeypot & Friendly Captcha (spam protection)
  - Focal Point (image cropping)
  - Easy Breadcrumb
  - Token

For a complete list of dependencies, see `composer.json`.

### Getting Help

- **Drupal Docs**: [https://www.drupal.org/docs](https://www.drupal.org/docs)
- **Event Platform**: [https://www.drupal.org/project/event_platform](https://www.drupal.org/project/event_platform)
- **Event Horizon Theme**: [https://www.drupal.org/project/event_horizon](https://www.drupal.org/project/event_horizon)
- **Drupal Chat**: [https://www.drupal.org/node/314178](https://www.drupal.org/node/314178)

---

**Happy Building! 🚀**

