# My Cybersecurity Blog

Welcome to my cybersecurity blog! This site will feature various write-ups on security topics, tools, and techniques. Click on the headlines below to read the full articles.

## Blog Write-Ups

- [Privilege Escalation Techniques: Kerberoasting and DCSync](posts/privilege-escalation.md)
- [Dumping Credentials with Mimikatz](posts/credentials-dump.md)
- [Understanding Persistence: Scheduled Tasks and New Users](posts/persistence-techniques.md)
- [Network Enumeration with Netstat and Arp](posts/network-enumeration.md)
- [Using Certutil for Payload Downloads](posts/certutil-techniques.md)

## Getting Started

This site is powered by the Hacker theme on GitHub Pages. If you're interested in using this theme for your own blog, see the instructions below.

## Usage

1. Add the following to your site's `_config.yml` to use the Hacker theme:

    ```yml
    remote_theme: pages-themes/hacker@v0.2.0
    plugins:
      - jekyll-remote-theme
    ```

2. To preview the site locally, add the following to your `Gemfile`:

    ```ruby
    gem "github-pages", group: :jekyll_plugins
    ```

## Customization

### Configuration

Update your `_config.yml` to set the title and description:

```yml
title: "My Cybersecurity Blog"
description: "A collection of write-ups on security techniques"
