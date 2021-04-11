---
title: "Build WAZUH System"
tags: [wazuh, hids, "security infra"]
categories: [ids, infra]
---

1. [Introduce](#introduce)
2. [Base System](#base-system)

## Introduce

Wazuh is Opensource Host-Based IDS system. It works with Elastic search and Kibana and latest version is 4.1.4. Official web site is [here](https://wazuh.com/).

In the wazuh system, manager is mean server and agent is mean client. Wazuh manager is supported various linux distribution, you can check system requirement from [here](https://documentation.wazuh.com/current/installation-guide/requirements.html#requirements).

Wazuh agent is support linux, windows, macOS and etc. For install agent, it need to 0.1GB RAM so you can run that at mostly system.

## Base System

I will try to installed wazuh manager on "**Fedora 33 Server Edition**" and agent system is set up as a "**windows 10**".

### Server Installation

Install is much easily if you use "Unattended installation". Just copy script and paste your terminal, it's done. Here is the [link](https://documentation.wazuh.com/current/installation-guide/open-distro/all-in-one-deployment/unattended-installation.html#installing-wazuh) for installation by one-command. Successfully install is ended, you can access the wazuh manger pages "https://(server ip)", default credential is "admin:admin".

![Wazuh Login](https://github.com/Jun-Project-LAB/Jun-Project-LAB.github.io/blob/main/_image/wazuh_login.png?raw=true)

Changing admin password is much as important. Click the profile located right top, and select "Rest password".

![Wazuh_Change_password](https://raw.githubusercontent.com/Jun-Project-LAB/Jun-Project-LAB.github.io/main/_image/wazuh_change_password.png)

I tried that but error occurred like this [article](https://stackoverflow.com/questions/65401755/wazuh-how-to-change-admin-password-for-web-interface). It looks like donse't allow this way.


