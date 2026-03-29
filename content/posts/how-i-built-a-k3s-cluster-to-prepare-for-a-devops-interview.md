---
title: "How I Built a K3s Home Lab to Prepare for a DevOps Interview"
description: "I had two weeks before a technical interview at an aerospace startup. Instead of just reading documentation, I built a 3-node K3s Kubernetes cluster from scratch using Terraform and Ansible. Here's what I built, what broke, and what I learned."
date: 2026-03-30T00:51:21+02:00
tags: ["kubernetes", "k3s", "terraform", "ansible", "devops", "homelab", "linux"]
author: "Michael Quinton"
draft: false
---

## Why I Built This

I had a technical interview coming up at a Berlin aerospace startup. The role was Junior DevOps Engineer and the job description was clear — strong Linux, Bash, Git, Ansible, and CI/CD.

I could have spent two weeks reading documentation. Instead I decided to build something real.

The idea was simple: if I'm going to talk about Terraform and Ansible in an interview, I should actually have used them recently. Not six months ago. This week.

So I built a 3-node K3s Kubernetes cluster on my local machine using KVM, Terraform, and Ansible. From scratch. In two weeks.

## The Architecture

- **Host:** Ubuntu 24.04 running KVM/libvirt
- **Nodes:** 1 master, 2 workers — all Debian 12 cloud images
- **Networking:** NAT network with static IP assignments via DHCP MAC binding
- **Provisioning:** Terraform with the dmacvicar/libvirt provider
- **Configuration:** Ansible with proper role structure
- **CI/CD:** GitLab CI pipeline for validation and linting

## The Terraform Setup

The core of the infrastructure is a `locals` block defining all three nodes:
```hcl
locals {
  k3s = {
    "k3s-master-1" = { os_code_name = "bookworm", octetIP = "201", vcpu = 2, memoryMB = 2048, incGB = 20 },
    "k3s-worker-1" = { os_code_name = "bookworm", octetIP = "202", vcpu = 1, memoryMB = 1536, incGB = 20 },
    "k3s-worker-2" = { os_code_name = "bookworm", octetIP = "203", vcpu = 1, memoryMB = 1536, incGB = 20 },
  }
}
```

Using `for_each` over this local means I define the node configuration once and loop over it. No copy-pasting three identical resource blocks.

MAC addresses are generated dynamically from the IP octet:
```hcl
mac = format("52:54:00:00:00:%02x", each.value.octetIP)
```

Nodes use backing store volumes rather than full image copies — saves disk space significantly.

## The Ansible Structure

Rather than flat playbooks I structured this as proper Ansible roles:
```
ansible/
├── ansible.cfg
├── host_prep.yml
├── k3s-setup.yml
├── group_vars/
│   ├── all.yml
│   ├── k3s_masters.yml
│   └── k3s_workers.yml
├── inventory/
│   ├── production/
│   └── staging/
└── roles/
    ├── common/
    ├── k3s_master/
    └── k3s_worker/
```

Separate inventories for production and staging. Run against either:
```bash
ansible-playbook -i inventory/staging k3s-setup.yml
ansible-playbook -i inventory/production k3s-setup.yml
```

## The CI/CD Pipeline

I added a GitLab CI pipeline that validates and lints on every push:
```yaml
stages:
  - validate
  - lint

terraform-validate:
  stage: validate
  image: hashicorp/terraform:latest
  script:
    - cd terraform
    - terraform init -backend=false
    - terraform validate

ansible-lint:
  stage: lint
  image: python:3.11
  before_script:
    - pip install ansible ansible-lint
  script:
    - ansible-lint ansible/

bash-lint:
  stage: lint
  image: koalaman/shellcheck-alpine:latest
  script:
    - shellcheck ansible/roles/common/files/health_check.sh
```

Three jobs. Terraform validation, Ansible linting, and Shellcheck on the Bash scripts. If any of them fail the pipeline fails.

## What Broke

**The runner typo** — my first GitHub Actions pipeline sat waiting for 24 hours before I realised I'd written `unbuntu-latest` instead of `ubuntu-latest`. One letter. Classic.

**set -euo pipefail** — I learned the hard way that `set -u` treats unbound variables as errors. A script that looked fine would crash immediately if called without arguments because `$1` didn't exist. Fix: use `${1:-}` as a default.

**Ansible and libvirtd** — easy to forget to start and enable `libvirtd` after installing it. Nothing works without it and it fails silently in confusing ways.

## What I Learned

Building something real is completely different from reading about it.

I understood `for_each` in Terraform from documentation. But it was only when I was actually looping over three VM definitions and generating MAC addresses dynamically that I really understood why it matters.

Same with Ansible roles. The folder structure makes complete sense when you're actually separating master and worker configuration across `group_vars` and seeing how `site.yml` ties it all together.

The CI pipeline taught me what linting actually means in practice — not just a concept but a real thing that catches real mistakes. Like the runner name typo.

## The GitHub Repo

Everything is on GitHub if you want to take a look or use it as a starting point:

[github.com/michael-quinton/k3s-at-home](https://github.com/michael-quinton/k3s-at-home)

---

*Built in Berlin, March 2026.*