---
title: "Four Things I'd Tell Myself Before Adopting GitOps"
description: "Practical lessons on rolling out ArgoCD and GitOps workflows across multiple environments without the migration turning into a mess."
pubDate: 2026-01-15
draft: true
tags: ["GitOps", "ArgoCD", "Kubernetes"]
---

GitOps sounds simple on a slide: your Git repo is the source of truth, a controller
reconciles the cluster to match it, and drift just... stops happening. Rolling it out
in a real environment with multiple teams and existing pipelines is a bit messier than
that. A few things worth knowing before you start.

## 1. Decide on a repo strategy before you touch ArgoCD

The two common patterns are a mono-repo with folders per environment, or separate repos
per environment/team. Mono-repo is easier to audit and diff across environments, but it
concentrates access control in one place. Separate repos scale better for team autonomy
but make it harder to see "what's different between staging and prod" at a glance. Pick
one deliberately — retrofitting this later means rewriting every Application manifest.

## 2. Drift detection needs a policy, not just a dashboard

ArgoCD will happily show you every out-of-sync resource, but that's only useful if
someone owns the response. Decide up front: does the controller auto-heal drift
immediately, or does it flag it for a human to review first? Auto-heal is great for
config the team fully controls; it's dangerous for anything that other tooling (autoscalers,
operators, admission controllers) might legitimately mutate at runtime.

## 3. Secrets don't belong in the Git repo, even encrypted

Sealed Secrets and SOPS are fine tools, but the simplest long-term answer is usually to
keep secrets out of Git entirely and let ArgoCD reference an external secret manager at
sync time. It's less convenient on day one and saves you a very bad day later.

## 4. Progressive delivery is a separate decision from GitOps itself

Blue-green and canary rollouts (via Argo Rollouts, Istio traffic splitting, or similar)
solve a different problem than GitOps does. GitOps gets the right manifest applied;
progressive delivery controls how traffic shifts to the new version. Bolting both on at
once makes it hard to tell which layer broke when something goes wrong — get plain
GitOps stable first, then layer in traffic shifting.

None of this is a reason to avoid GitOps — the audit trail and rollback story alone are
worth it. Just go in expecting it to be an operating model change, not a tool install.
