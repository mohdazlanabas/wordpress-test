# WordPress Test Project

## Table of Contents

- [Project Overview](#project-overview)
- [Platform Information](#platform-information)
- [Project Purpose](#project-purpose)
- [Architecture and Structure](#architecture-and-structure)
- [Repository Configuration](#repository-configuration)
- [Installation and Setup](#installation-and-setup)
- [Deployment](#deployment)
- [File Structure](#file-structure)
- [WordPress Components](#wordpress-components)
- [Git Strategy](#git-strategy)
- [Security Considerations](#security-considerations)
- [Maintenance and Updates](#maintenance-and-updates)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

---

## Project Overview

This repository contains a **WordPress test environment** deployed on **Google Cloud Platform's App Engine**. It serves as a development and testing platform for WordPress-based web applications, providing a cloud-hosted instance that can be used for experimentation, plugin/theme development, or client demonstrations.

### Live Instance

- **Platform**: Google Cloud App Engine
- **URL**: http://34.124.169.139/
- **Environment**: Test/Development

---

## Platform Information

### Google Cloud App Engine

This WordPress installation is hosted on Google Cloud Platform's App Engine, a fully managed serverless platform that allows applications to scale automatically based on traffic demands. App Engine handles infrastructure management, including:

- **Automatic Scaling**: Resources scale up or down based on traffic patterns
- **Load Balancing**: Distributes incoming requests across multiple instances
- **Security**: Built-in DDoS protection and SSL/TLS support
- **Monitoring**: Integrated logging and monitoring through Google Cloud Console
- **Zero Server Management**: No need to manage VMs, containers, or operating systems

### Benefits of App Engine for WordPress

1. **High Availability**: Automatic failover and redundancy
2. **Performance**: Global CDN integration and edge caching capabilities
3. **Cost Efficiency**: Pay only for resources actually consumed
4. **Easy Deployment**: Simple deployment process via gcloud CLI
5. **Integration**: Seamless integration with other Google Cloud services (Cloud SQL, Cloud Storage, etc.)

---

## Project Purpose

This WordPress test project is designed to serve multiple purposes:

### 1. Development and Testing Environment
- Test WordPress core functionality in a cloud environment
- Experiment with plugins and themes before production deployment
- Validate compatibility between different WordPress components
- Test performance under various load conditions

### 2. Client Demonstrations
- Showcase WordPress capabilities to potential clients
- Provide a live demo environment for stakeholders
- Test custom development before client approval

### 3. Learning and Experimentation
- Learn WordPress development best practices
- Experiment with Google Cloud Platform services
- Test deployment pipelines and CI/CD workflows
- Understand WordPress security and optimization techniques

### 4. Plugin and Theme Development
- Develop and test custom WordPress plugins
- Create and refine custom themes
- Test third-party plugin integrations
- Debug and troubleshoot WordPress issues in a safe environment

---

## Architecture and Structure

### WordPress Installation Type

This is a **selective WordPress installation** that uses Git version control for specific components while excluding the WordPress core files. This architecture provides several advantages:

#### Tracked Components (Version Controlled)
- Custom themes in `wp-content/themes/`
- Custom and third-party plugins in `wp-content/plugins/`
- Configuration files (`.htaccess`, `.gitignore`)
- Project documentation and utilities

#### Excluded Components (Not Tracked)
- WordPress core files (`/wp-admin/`, `/wp-includes/`, core PHP files)
- Configuration with secrets (`wp-config.php`)
- Uploaded media files (`/wp-content/uploads/`)
- Cache and temporary files
- Database backups and logs

### Why This Approach?

This selective version control strategy offers several benefits:

1. **Smaller Repository Size**: By excluding WordPress core and uploads, the repository remains lightweight
2. **Security**: Sensitive configuration files with database credentials are never committed
3. **Flexibility**: WordPress core can be updated independently without merge conflicts
4. **Focus**: Version control focuses on custom development work (themes and plugins)
5. **Clean History**: Git history remains clean and relevant to actual development work

---

## Repository Configuration

### .gitignore Strategy

The project uses a comprehensive [.gitignore](.gitignore) file that implements a whitelist approach:

```
# WordPress core (don't track - install fresh)
/wp-admin/
/wp-includes/
/wp-*.php
/index.php
/license.txt
/readme.html
/xmlrpc.php

# Config (contains secrets)
wp-config.php

# Uploads (too large for git)
/wp-content/uploads/

# Cache and temp
/wp-content/cache/
/wp-content/upgrade/
/wp-content/backup*/
*.log

# OS files
.DS_Store
Thumbs.db

# Keep themes and plugins
!/wp-content/
!/wp-content/themes/
!/wp-content/plugins/
```

This configuration ensures that:
- WordPress core files are excluded from version control
- Custom themes and plugins are tracked
- Sensitive configuration is protected
- Temporary and generated files are ignored
- The repository remains clean and focused

### .htaccess Configuration

The project includes a standard WordPress [.htaccess](.htaccess) file that enables:

- **Pretty Permalinks**: SEO-friendly URLs without query strings
- **URL Rewriting**: Routes all requests through WordPress's index.php
- **HTTP Authorization**: Preserves authorization headers for API requests
- **404 Handling**: Proper handling of non-existent files

Key features of the `.htaccess` configuration:

```apache
RewriteEngine On
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
RewriteBase /
RewriteRule ^index\.php$ - [L]
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule . /index.php [L]
```

---

## Installation and Setup

### Prerequisites

Before setting up this WordPress installation, ensure you have:

1. **PHP Environment**
   - PHP 7.4 or higher (PHP 8.0+ recommended)
   - MySQL 5.7+ or MariaDB 10.3+ database
   - Apache or Nginx web server with mod_rewrite enabled

2. **Required PHP Extensions**
   - mysqli or PDO extension
   - GD library or ImageMagick
   - mbstring extension
   - xml and xmlrpc extensions
   - curl extension
   - zip extension
   - intl extension (recommended)

3. **Google Cloud SDK** (for deployment)
   - gcloud CLI tool installed and configured
   - Active Google Cloud Platform account
   - Project created in GCP Console

4. **Development Tools**
   - Git for version control
   - Text editor or IDE (VSCode, PHPStorm, Sublime Text, etc.)
   - Command line/terminal access

### Local Development Setup

#### Step 1: Clone the Repository

```bash
git clone <repository-url>
cd wordpress-test
```

#### Step 2: Install WordPress Core

Since WordPress core files are not tracked in Git, you'll need to download and install them:

```bash
# Download latest WordPress
curl -O https://wordpress.org/latest.tar.gz

# Extract core files
tar -xzf latest.tar.gz

# Move core files to project root (excluding wp-content)
rsync -av --exclude 'wp-content' wordpress/* ./

# Clean up
rm -rf wordpress latest.tar.gz
```

#### Step 3: Create Database

Create a MySQL database for your WordPress installation:

```sql
CREATE DATABASE wordpress_test CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON wordpress_test.* TO 'wordpress_user'@'localhost';
FLUSH PRIVILEGES;
```

#### Step 4: Configure WordPress

Create a `wp-config.php` file from the sample:

```bash
cp wp-config-sample.php wp-config.php
```

Edit `wp-config.php` with your database credentials:

```php
define( 'DB_NAME', 'wordpress_test' );
define( 'DB_USER', 'wordpress_user' );
define( 'DB_PASSWORD', 'secure_password' );
define( 'DB_HOST', 'localhost' );
```

Generate unique authentication keys and salts from [WordPress Secret Key Generator](https://api.wordpress.org/secret-key/1.1/salt/) and add them to `wp-config.php`.

#### Step 5: Set File Permissions

```bash
# Set proper ownership (adjust username as needed)
sudo chown -R www-data:www-data .

# Set directory permissions
find . -type d -exec chmod 755 {} \;

# Set file permissions
find . -type f -exec chmod 644 {} \;
```

#### Step 6: Access WordPress Installer

Navigate to `http://localhost/` (or your local development URL) and complete the WordPress installation wizard:

1. Select your language
2. Enter site title and admin credentials
3. Configure admin email address
4. Choose search engine visibility settings

---

## Deployment

### Deploying to Google Cloud App Engine

#### Step 1: Configure App Engine

Create an `app.yaml` file in the project root:

```yaml
runtime: php81
env: standard

handlers:
- url: /wp-content/(.+)
  static_files: wp-content/\1
  upload: wp-content/.*

- url: /(.+\.(ico|jpg|jpeg|png|gif|css|js|woff|woff2|ttf|svg))$
  static_files: \1
  upload: (.+\.(ico|jpg|jpeg|png|gif|css|js|woff|woff2|ttf|svg))$

- url: /.*
  script: auto
```

#### Step 2: Configure Cloud SQL (Optional but Recommended)

For production use, set up Cloud SQL:

```bash
# Create Cloud SQL instance
gcloud sql instances create wordpress-db \
  --database-version=MYSQL_8_0 \
  --tier=db-f1-micro \
  --region=us-central1

# Create database
gcloud sql databases create wordpress --instance=wordpress-db

# Create user
gcloud sql users create wordpress-user \
  --instance=wordpress-db \
  --password=SECURE_PASSWORD
```

#### Step 3: Deploy to App Engine

```bash
# Authenticate with Google Cloud
gcloud auth login

# Set your project
gcloud config set project YOUR_PROJECT_ID

# Deploy the application
gcloud app deploy

# Open the deployed application
gcloud app browse
```

#### Step 4: Post-Deployment Configuration

After deployment:

1. Update WordPress site URL in the database
2. Configure Cloud Storage for media uploads (recommended)
3. Set up Cloud CDN for static assets
4. Configure SSL/TLS certificate
5. Set up monitoring and logging

### Continuous Deployment

For automated deployments, consider setting up:

- **Cloud Build**: Automated builds triggered by Git commits
- **GitHub Actions**: CI/CD pipeline integration
- **GitLab CI/CD**: Automated testing and deployment

---

## File Structure

```
wordpress-test/
├── .git/                          # Git version control directory
├── .gitignore                     # Git ignore rules
├── .htaccess                      # Apache rewrite rules
├── info.php                       # PHP information script (phpinfo)
├── url.md                         # Deployment URL documentation
├── README.md                      # This comprehensive documentation
│
├── wp-content/                    # WordPress content directory (tracked)
│   ├── index.php                  # Directory security file
│   │
│   ├── plugins/                   # WordPress plugins
│   │   ├── index.php              # Directory security file
│   │   ├── akismet/               # Akismet anti-spam plugin
│   │   └── hello.php              # Hello Dolly plugin (sample)
│   │
│   └── themes/                    # WordPress themes
│       ├── index.php              # Directory security file
│       ├── twentytwentyfive/      # WordPress Twenty Twenty-Five theme
│       ├── twentytwentyfour/      # WordPress Twenty Twenty-Four theme
│       └── twentytwentythree/     # WordPress Twenty Twenty-Three theme
│
├── wp-admin/                      # WordPress admin (not tracked)
├── wp-includes/                   # WordPress core libraries (not tracked)
├── wp-config.php                  # WordPress configuration (not tracked)
└── [other WordPress core files]   # WordPress core files (not tracked)
```

### Key Files and Directories

#### Tracked Files

**[.gitignore](.gitignore)**
- Defines which files and directories are excluded from version control
- Implements a whitelist approach for WordPress components
- Protects sensitive configuration and temporary files

**[.htaccess](.htaccess)**
- Apache web server configuration
- Enables WordPress permalink functionality
- Configures URL rewriting rules
- Sets up proper request routing

**[info.php](info.php)**
- PHP configuration diagnostic script
- Displays comprehensive PHP environment information
- Useful for debugging and verifying server configuration
- **Security Note**: Should be removed or restricted in production

**[url.md](url.md)**
- Documents the deployment platform and URL
- Quick reference for accessing the live site
- Contains platform-specific information

**wp-content/plugins/**
- Contains WordPress plugins (tracked in Git)
- Currently includes:
  - **Akismet**: Anti-spam plugin for comment filtering
  - **Hello Dolly**: Sample plugin demonstrating WordPress plugin structure

**wp-content/themes/**
- Contains WordPress themes (tracked in Git)
- Currently includes:
  - **Twenty Twenty-Five**: Latest default WordPress theme
  - **Twenty Twenty-Four**: Previous default theme
  - **Twenty Twenty-Three**: Older default theme

#### Not Tracked (Excluded from Git)

**wp-config.php**
- WordPress configuration file
- Contains database credentials and security keys
- **Never commit this file** - it contains sensitive information

**wp-admin/**
- WordPress administration interface
- Part of WordPress core, updated with WordPress releases

**wp-includes/**
- WordPress core libraries and functions
- Updated with WordPress core updates

**wp-content/uploads/**
- User-uploaded media files
- Can grow very large, not suitable for Git
- Should be backed up separately or stored in cloud storage

---

## WordPress Components

### Included Plugins

#### Akismet Anti-Spam

**Purpose**: Protects WordPress sites from comment and trackback spam

**Features**:
- Automatic spam detection using machine learning
- Comment moderation tools
- Spam statistics and reporting
- Integration with WordPress.com accounts
- REST API for programmatic access

**Location**: `wp-content/plugins/akismet/`

**Key Files**:
- `akismet.php`: Main plugin file
- `class.akismet.php`: Core Akismet functionality
- `class.akismet-admin.php`: Admin interface
- `class.akismet-widget.php`: Dashboard widget
- `class.akismet-rest-api.php`: REST API endpoints

**Configuration**: Requires an Akismet API key from WordPress.com

#### Hello Dolly

**Purpose**: Sample plugin demonstrating WordPress plugin architecture

**Features**:
- Displays random lyrics from "Hello, Dolly" in the admin header
- Minimal plugin demonstrating hooks and filters
- Educational tool for plugin developers

**Location**: `wp-content/plugins/hello.php`

**Note**: This plugin is typically disabled or removed in production environments, but serves as a useful reference for plugin development.

### Included Themes

#### Twenty Twenty-Five

**Location**: `wp-content/themes/twentytwentyfive/`

**Description**: The latest default WordPress theme, featuring:
- Block-based full-site editing (FSE)
- Modern, responsive design
- Extensive customization options
- Pattern library for quick page building
- Accessibility-ready (WCAG 2.1 Level AA compliant)
- Performance optimized

**Use Cases**:
- Modern, content-focused websites
- Blogs and personal sites
- Portfolio websites
- Business sites with minimal complexity

#### Twenty Twenty-Four

**Location**: `wp-content/themes/twentytwentyfour/`

**Description**: Previous default theme with:
- Block-based design system
- Flexible layout options
- Multiple style variations
- Template parts and patterns
- Responsive typography

#### Twenty Twenty-Three

**Location**: `wp-content/themes/twentytwentythree/`

**Description**: Earlier default theme featuring:
- Introduction to block themes
- Simplified customization
- Clean, minimalist design
- Educational value for theme developers

### Adding Custom Plugins and Themes

#### Installing a Plugin

**Via WordPress Admin**:
1. Navigate to `Plugins > Add New`
2. Search for the desired plugin
3. Click "Install Now"
4. Activate the plugin
5. Commit the new plugin to Git:
   ```bash
   git add wp-content/plugins/new-plugin-name/
   git commit -m "Add [plugin name] plugin"
   git push
   ```

**Via Manual Upload**:
1. Download the plugin ZIP file
2. Extract to `wp-content/plugins/`
3. Activate via WordPress admin
4. Commit to Git

**Via WP-CLI** (if available):
```bash
wp plugin install plugin-name --activate
git add wp-content/plugins/plugin-name/
git commit -m "Add [plugin name] plugin"
```

#### Installing a Theme

**Via WordPress Admin**:
1. Navigate to `Appearance > Themes > Add New`
2. Search for the desired theme
3. Click "Install" then "Activate"
4. Commit to Git:
   ```bash
   git add wp-content/themes/new-theme-name/
   git commit -m "Add [theme name] theme"
   git push
   ```

**Via Manual Upload**:
1. Download the theme ZIP file
2. Extract to `wp-content/themes/`
3. Activate via `Appearance > Themes`
4. Commit to Git

---

## Git Strategy

### Branching Strategy

This project can implement various Git workflows depending on team size and complexity:

#### Simple Workflow (Recommended for Small Teams)

```
main (production-ready code)
  ├── feature/new-plugin
  ├── feature/theme-customization
  └── hotfix/security-patch
```

**Branches**:
- `main`: Production-ready code, deployed to App Engine
- `feature/*`: New features or enhancements
- `hotfix/*`: Urgent fixes for production issues

#### Git Flow (For Larger Teams)

```
main (production)
  └── develop (integration)
      ├── feature/contact-form
      ├── feature/new-homepage
      └── release/v1.2.0
```

**Branches**:
- `main`: Production releases only
- `develop`: Integration branch for features
- `feature/*`: Individual features
- `release/*`: Release preparation
- `hotfix/*`: Production hotfixes

### Commit Message Conventions

Use clear, descriptive commit messages following this format:

```
<type>: <subject>

<body>

<footer>
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Formatting, missing semicolons, etc.
- `refactor`: Code restructuring
- `test`: Adding tests
- `chore`: Maintenance tasks

**Examples**:

```
feat: Add contact form plugin for homepage

- Installed Contact Form 7 plugin
- Configured form fields for customer inquiries
- Styled form to match site theme

Resolves #123
```

```
fix: Resolve permalink 404 errors

Updated .htaccess rewrite rules to properly handle
custom post type permalinks.

Fixes #456
```

### Best Practices

1. **Commit Frequently**: Make small, focused commits
2. **Write Descriptive Messages**: Explain the "why" not just the "what"
3. **Review Before Committing**: Use `git diff` to review changes
4. **Pull Before Push**: Always pull latest changes before pushing
5. **Use Branches**: Never commit directly to `main`
6. **Tag Releases**: Use Git tags for version releases

### Workflow Example

```bash
# Create a new feature branch
git checkout -b feature/add-gallery-plugin

# Make changes (install plugin, configure, etc.)
# ...

# Stage and commit changes
git add wp-content/plugins/gallery-plugin/
git commit -m "feat: Add responsive gallery plugin

- Installed NextGEN Gallery plugin
- Configured thumbnail sizes
- Created sample gallery for portfolio section"

# Push to remote
git push origin feature/add-gallery-plugin

# Create pull request for review
# After approval, merge to main
git checkout main
git merge feature/add-gallery-plugin
git push origin main

# Deploy to App Engine
gcloud app deploy
```

---

## Security Considerations

### Configuration Security

#### wp-config.php Protection

The `wp-config.php` file is **never committed to Git** because it contains:
- Database credentials
- Authentication keys and salts
- Security tokens
- Environment-specific configuration

**Best Practices**:
1. Use environment variables for sensitive data
2. Generate unique keys using [WordPress Secret Key Generator](https://api.wordpress.org/secret-key/1.1/salt/)
3. Restrict file permissions: `chmod 600 wp-config.php`
4. Add security constants:
   ```php
   define('DISALLOW_FILE_EDIT', true);
   define('FORCE_SSL_ADMIN', true);
   define('WP_AUTO_UPDATE_CORE', 'minor');
   ```

### File Permissions

Proper file permissions are critical for security:

```bash
# Directories
find . -type d -exec chmod 755 {} \;

# Files
find . -type f -exec chmod 644 {} \;

# wp-config.php (more restrictive)
chmod 600 wp-config.php

# .htaccess
chmod 644 .htaccess
```

### Security Headers

Add security headers to `.htaccess`:

```apache
# Security Headers
Header set X-Content-Type-Options "nosniff"
Header set X-Frame-Options "SAMEORIGIN"
Header set X-XSS-Protection "1; mode=block"
Header set Referrer-Policy "strict-origin-when-cross-origin"
```

### Security Plugins (Recommended)

Consider installing security plugins:

1. **Wordfence Security**: Firewall and malware scanner
2. **iThemes Security**: Comprehensive security hardening
3. **Sucuri Security**: Security auditing and monitoring
4. **All In One WP Security**: User-friendly security plugin

### info.php Security Warning

The `info.php` file exposes detailed PHP configuration information. This is useful for development but **should be removed or password-protected in production**:

**Option 1: Remove**
```bash
rm info.php
```

**Option 2: Restrict Access**
Add to `.htaccess`:
```apache
<Files "info.php">
    AuthType Basic
    AuthName "Restricted Access"
    AuthUserFile /path/to/.htpasswd
    Require valid-user
</Files>
```

### Regular Security Audits

Perform regular security audits:

1. **Update WordPress Core**: Keep WordPress updated to latest version
2. **Update Plugins**: Regularly update all plugins
3. **Update Themes**: Keep themes current
4. **Review User Accounts**: Remove inactive users, enforce strong passwords
5. **Monitor Logs**: Check error logs for suspicious activity
6. **Backup Regularly**: Maintain current backups
7. **Scan for Malware**: Use security plugins to scan for malware

---

## Maintenance and Updates

### Updating WordPress Core

#### Via WordPress Admin

1. Navigate to `Dashboard > Updates`
2. Click "Update Now" for WordPress core
3. Test the site after update
4. **Note**: Core files are not tracked in Git, so no commit needed

#### Via WP-CLI

```bash
# Check for updates
wp core check-update

# Update WordPress core
wp core update

# Update database if needed
wp core update-db
```

### Updating Plugins

#### Via WordPress Admin

1. Navigate to `Plugins > Installed Plugins`
2. Click "Update Now" for each plugin
3. Test functionality after updates
4. Commit updated plugins to Git:
   ```bash
   git add wp-content/plugins/
   git commit -m "chore: Update plugins to latest versions

   - Updated Akismet to v5.3
   - Updated [other plugins]"
   git push
   ```

#### Via WP-CLI

```bash
# List plugin updates
wp plugin list --update=available

# Update all plugins
wp plugin update --all

# Update specific plugin
wp plugin update akismet
```

### Updating Themes

#### Via WordPress Admin

1. Navigate to `Appearance > Themes`
2. Click "Update Now" for each theme
3. Test theme functionality
4. Commit to Git:
   ```bash
   git add wp-content/themes/
   git commit -m "chore: Update themes to latest versions"
   git push
   ```

### Backup Strategy

#### What to Backup

1. **Database**: All WordPress content, settings, and user data
2. **wp-content/uploads/**: User-uploaded media files
3. **wp-content/plugins/**: Custom and third-party plugins (also in Git)
4. **wp-content/themes/**: Custom themes (also in Git)
5. **wp-config.php**: Configuration file (store securely, not in Git)

#### Backup Methods

**Manual Backup**:
```bash
# Database backup
mysqldump -u username -p database_name > backup_$(date +%Y%m%d).sql

# File backup
tar -czf backup_$(date +%Y%m%d).tar.gz wp-content/uploads/ wp-config.php
```

**Plugin-Based Backup**:
- **UpdraftPlus**: Scheduled backups to cloud storage
- **BackupBuddy**: Comprehensive backup solution
- **Duplicator**: Full site migration and backup

**Google Cloud Backup**:
- Cloud SQL automated backups
- Cloud Storage for file backups
- Snapshot scheduling for persistent disks

### Monitoring

#### Performance Monitoring

- **Google Cloud Monitoring**: Track App Engine metrics
- **Query Monitor**: WordPress plugin for debugging
- **New Relic**: Application performance monitoring
- **Jetpack**: Site statistics and monitoring

#### Uptime Monitoring

- **Google Cloud Monitoring**: Uptime checks
- **UptimeRobot**: Free uptime monitoring
- **Pingdom**: Comprehensive uptime and performance monitoring
- **StatusCake**: Website monitoring and alerts

#### Error Monitoring

Enable WordPress debugging in development:

```php
// In wp-config.php (development only)
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', false);
```

Monitor error logs:
```bash
tail -f wp-content/debug.log
```

---

## Troubleshooting

### Common Issues and Solutions

#### Issue: 404 Errors on All Pages Except Homepage

**Cause**: Permalink rewrite rules not working

**Solutions**:

1. **Check .htaccess file exists and is readable**
   ```bash
   ls -la .htaccess
   ```

2. **Regenerate .htaccess**
   - Go to `Settings > Permalinks`
   - Click "Save Changes" without making changes

3. **Verify mod_rewrite is enabled** (Apache)
   ```bash
   apache2ctl -M | grep rewrite
   ```

4. **Check file permissions**
   ```bash
   chmod 644 .htaccess
   ```

#### Issue: White Screen of Death (WSOD)

**Cause**: PHP fatal error, memory limit, or plugin/theme conflict

**Solutions**:

1. **Enable debugging**
   ```php
   define('WP_DEBUG', true);
   define('WP_DEBUG_LOG', true);
   ```

2. **Increase memory limit**
   ```php
   define('WP_MEMORY_LIMIT', '256M');
   ```

3. **Disable plugins**
   ```bash
   # Rename plugins directory
   mv wp-content/plugins wp-content/plugins-disabled
   ```

4. **Switch to default theme**
   - Rename current theme directory
   - WordPress will auto-activate default theme

5. **Check error logs**
   ```bash
   tail -f wp-content/debug.log
   ```

#### Issue: Database Connection Error

**Cause**: Incorrect database credentials or database server unavailable

**Solutions**:

1. **Verify database credentials in wp-config.php**
   ```php
   define('DB_NAME', 'correct_database_name');
   define('DB_USER', 'correct_username');
   define('DB_PASSWORD', 'correct_password');
   define('DB_HOST', 'correct_host');
   ```

2. **Test database connection**
   ```bash
   mysql -u username -p -h hostname database_name
   ```

3. **Check database server status**
   ```bash
   # For local MySQL
   sudo systemctl status mysql

   # For Cloud SQL
   gcloud sql instances describe instance-name
   ```

4. **Verify database exists**
   ```sql
   SHOW DATABASES;
   ```

#### Issue: Upload Directory Not Writable

**Cause**: Incorrect file permissions on wp-content/uploads

**Solutions**:

1. **Create uploads directory if missing**
   ```bash
   mkdir -p wp-content/uploads
   ```

2. **Set proper permissions**
   ```bash
   sudo chown -R www-data:www-data wp-content/uploads
   chmod 755 wp-content/uploads
   ```

3. **For App Engine**: Configure Cloud Storage bucket for uploads

#### Issue: Plugin/Theme Installation Fails

**Cause**: FTP credentials requested or permission issues

**Solutions**:

1. **Add direct filesystem method to wp-config.php**
   ```php
   define('FS_METHOD', 'direct');
   ```

2. **Set proper ownership**
   ```bash
   sudo chown -R www-data:www-data wp-content
   ```

3. **Install manually via SFTP/SSH**

#### Issue: Slow Performance

**Cause**: Multiple factors (plugins, theme, hosting, database)

**Solutions**:

1. **Install caching plugin**
   - WP Super Cache
   - W3 Total Cache
   - WP Rocket

2. **Optimize database**
   ```bash
   wp db optimize
   ```

3. **Enable object caching** (Memcached/Redis)

4. **Optimize images**
   - Install image optimization plugin (Smush, ShortPixel)
   - Use WebP format

5. **Enable CDN**
   - Configure Cloud CDN on App Engine
   - Use Cloudflare or similar service

6. **Review plugin usage**
   - Disable unnecessary plugins
   - Find lightweight alternatives

#### Issue: App Engine Deployment Fails

**Cause**: Configuration errors, quota exceeded, or permission issues

**Solutions**:

1. **Check app.yaml syntax**
   ```bash
   gcloud app deploy --verbosity=debug
   ```

2. **Verify project and permissions**
   ```bash
   gcloud config list
   gcloud projects get-iam-policy PROJECT_ID
   ```

3. **Check quota limits**
   - Visit GCP Console > IAM & Admin > Quotas

4. **Review deployment logs**
   ```bash
   gcloud app logs tail -s default
   ```

### Getting Help

#### WordPress Resources

- **WordPress Support Forums**: https://wordpress.org/support/
- **WordPress Codex**: https://codex.wordpress.org/
- **WordPress Developer Handbook**: https://developer.wordpress.org/
- **WordPress Stack Exchange**: https://wordpress.stackexchange.com/

#### Google Cloud Resources

- **App Engine Documentation**: https://cloud.google.com/appengine/docs
- **Cloud SQL Documentation**: https://cloud.google.com/sql/docs
- **GCP Support**: https://cloud.google.com/support
- **GCP Community**: https://www.googlecloudcommunity.com/

#### General Development Resources

- **Stack Overflow**: https://stackoverflow.com/questions/tagged/wordpress
- **GitHub Issues**: Report issues in this repository
- **PHP Documentation**: https://www.php.net/docs.php

---

## Additional Resources

### Recommended Plugins

#### Performance
- **WP Rocket**: Premium caching and performance plugin
- **Autoptimize**: Optimize CSS, JS, and HTML
- **Query Monitor**: Debug queries and performance
- **WP-Optimize**: Database cleanup and optimization

#### Security
- **Wordfence Security**: Firewall and malware scanner
- **iThemes Security**: Security hardening and monitoring
- **Sucuri Security**: Security auditing
- **SSL Insecure Content Fixer**: Fix mixed content warnings

#### SEO
- **Yoast SEO**: Comprehensive SEO plugin
- **Rank Math**: SEO optimization and analytics
- **All in One SEO**: SEO toolkit

#### Development
- **Query Monitor**: Development debugging
- **Debug Bar**: WordPress debugging
- **WP-CLI**: Command-line interface
- **Advanced Custom Fields**: Custom field management

#### Backup
- **UpdraftPlus**: Automated backups to cloud storage
- **BackWPup**: Complete backup solution
- **Duplicator**: Migration and backup

### Learning Resources

#### WordPress Development
- [WordPress Theme Handbook](https://developer.wordpress.org/themes/)
- [WordPress Plugin Handbook](https://developer.wordpress.org/plugins/)
- [WordPress REST API Handbook](https://developer.wordpress.org/rest-api/)
- [WordPress Coding Standards](https://developer.wordpress.org/coding-standards/)

#### Google Cloud Platform
- [App Engine PHP Quickstart](https://cloud.google.com/appengine/docs/standard/php/quickstart)
- [Cloud SQL for MySQL](https://cloud.google.com/sql/docs/mysql/)
- [Cloud Storage Documentation](https://cloud.google.com/storage/docs)
- [GCP Training](https://cloud.google.com/training/)

#### PHP Development
- [PHP The Right Way](https://phptherightway.com/)
- [PHP Manual](https://www.php.net/manual/en/)
- [Composer Documentation](https://getcomposer.org/doc/)
- [PHPUnit Documentation](https://phpunit.de/documentation.html)

### Development Tools

#### Code Editors/IDEs
- **VSCode**: Free, extensible editor with WordPress extensions
- **PHPStorm**: Professional PHP IDE with WordPress support
- **Sublime Text**: Lightweight, fast editor
- **Atom**: Hackable text editor

#### Useful VSCode Extensions
- WordPress Snippets
- PHP Intelephense
- PHP Debug
- EditorConfig
- GitLens
- Docker

#### Command-Line Tools
- **WP-CLI**: WordPress command-line interface
- **Composer**: PHP dependency manager
- **Git**: Version control
- **gcloud**: Google Cloud SDK
- **Node.js/npm**: For theme/plugin development with build tools

### Testing Tools

#### Local Development Environments
- **Local by Flywheel**: One-click local WordPress environment
- **XAMPP**: Cross-platform Apache, MySQL, PHP stack
- **MAMP**: Mac/Windows Apache, MySQL, PHP
- **Docker**: Containerized development environment
- **Laravel Valet**: Mac development environment

#### Browser Testing
- **Chrome DevTools**: Built-in browser debugging
- **Firefox Developer Tools**: Browser debugging and testing
- **BrowserStack**: Cross-browser testing platform
- **Lambda Test**: Browser compatibility testing

#### Performance Testing
- **Google PageSpeed Insights**: Performance and SEO analysis
- **GTmetrix**: Performance monitoring
- **Pingdom Tools**: Website speed test
- **WebPageTest**: Detailed performance analysis

---

## Contributing

### How to Contribute

If this is a team project, contributions are welcome! Please follow these guidelines:

1. **Fork the Repository**: Create your own fork
2. **Create a Branch**: Use descriptive branch names (`feature/add-gallery`, `fix/permalink-issue`)
3. **Make Changes**: Follow WordPress coding standards
4. **Test Thoroughly**: Test your changes locally
5. **Commit**: Use clear, descriptive commit messages
6. **Push**: Push to your fork
7. **Pull Request**: Create a PR with detailed description

### Code Standards

Follow WordPress coding standards:
- [PHP Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/php/)
- [HTML Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/html/)
- [CSS Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/css/)
- [JavaScript Coding Standards](https://developer.wordpress.org/coding-standards/wordpress-coding-standards/javascript/)

### Code Review Process

All changes should be reviewed before merging:
1. Code quality and standards compliance
2. Security considerations
3. Performance impact
4. Documentation updates
5. Testing coverage

---

## License

WordPress is released under the [GPL v2 (or later) license](https://wordpress.org/about/license/).

This project inherits the GPL license from WordPress. All custom code should also be GPL-compatible.

---

## Contact and Support

### Project Maintainers

- **Organization**: Gaia Digital Agency
- **Project**: WordPress Test Environment
- **Platform**: Google Cloud App Engine

### Getting Support

For issues related to:
- **WordPress Core**: Check [WordPress Support Forums](https://wordpress.org/support/)
- **Plugins/Themes**: Check individual plugin/theme support pages
- **Google Cloud**: Use [GCP Support](https://cloud.google.com/support)
- **This Project**: Create an issue in this repository

### Useful Links

- **Live Site**: http://34.124.169.139/
- **WordPress Admin**: http://34.124.169.139/wp-admin/
- **Google Cloud Console**: https://console.cloud.google.com/
- **WordPress.org**: https://wordpress.org/

---

## Changelog

### Version History

**Current Version**: Initial Setup

#### [Unreleased]
- Initial project setup
- WordPress core installation
- Default themes included (Twenty Twenty-Three, Twenty Twenty-Four, Twenty Twenty-Five)
- Default plugins included (Akismet, Hello Dolly)
- Deployed to Google Cloud App Engine
- Basic documentation created

#### Future Enhancements
- Custom theme development
- Additional plugin integration
- Performance optimization
- SEO configuration
- Analytics integration
- Enhanced security measures
- Automated backup system
- Staging environment setup
- CI/CD pipeline implementation

---

## Appendix

### Glossary

**App Engine**: Google Cloud Platform's fully managed serverless platform for hosting web applications

**Block Theme**: WordPress theme built using the block editor (Gutenberg) and full-site editing

**FSE (Full-Site Editing)**: WordPress feature allowing block-based editing of entire site layout

**Permalink**: Permanent URL structure for WordPress posts and pages

**Plugin**: Modular code package that extends WordPress functionality

**Theme**: Collection of templates and styles that define a WordPress site's appearance

**WP-CLI**: Official command-line interface for WordPress

**wp-config.php**: WordPress configuration file containing database credentials and settings

**wp-content**: Directory containing themes, plugins, and uploads

### Useful Commands Reference

#### WordPress (WP-CLI)

```bash
# Core management
wp core version                    # Check WordPress version
wp core update                     # Update WordPress core
wp core verify-checksums           # Verify core file integrity

# Plugin management
wp plugin list                     # List all plugins
wp plugin install plugin-name      # Install plugin
wp plugin activate plugin-name     # Activate plugin
wp plugin update --all             # Update all plugins

# Theme management
wp theme list                      # List all themes
wp theme activate theme-name       # Activate theme
wp theme update --all              # Update all themes

# Database management
wp db export backup.sql            # Export database
wp db import backup.sql            # Import database
wp db optimize                     # Optimize database
wp db repair                       # Repair database

# Cache management
wp cache flush                     # Flush object cache
wp rewrite flush                   # Flush rewrite rules

# User management
wp user list                       # List users
wp user create username email      # Create user
wp user update 1 --role=administrator  # Update user role
```

#### Google Cloud (gcloud)

```bash
# Authentication and configuration
gcloud auth login                  # Authenticate
gcloud config set project PROJECT_ID  # Set project
gcloud config list                 # Show configuration

# App Engine
gcloud app deploy                  # Deploy application
gcloud app browse                  # Open in browser
gcloud app logs tail               # View logs
gcloud app describe                # Show app details

# Cloud SQL
gcloud sql instances list          # List databases
gcloud sql instances describe DB   # Database details
gcloud sql databases list --instance=DB  # List databases
gcloud sql connect DB --user=root  # Connect to database
```

#### Git

```bash
# Basic commands
git status                         # Check status
git add .                          # Stage all changes
git commit -m "message"            # Commit changes
git push                           # Push to remote
git pull                           # Pull from remote

# Branching
git branch                         # List branches
git branch feature-name            # Create branch
git checkout feature-name          # Switch branch
git checkout -b feature-name       # Create and switch
git merge feature-name             # Merge branch

# History
git log                            # View commit history
git log --oneline                  # Compact history
git diff                           # Show changes
git show commit-hash               # Show commit details
```

---

**Last Updated**: 2026-01-04

**Document Version**: 1.0

**WordPress Version**: Latest (to be installed)

**PHP Version**: 8.1 (App Engine)

**Platform**: Google Cloud App Engine

