# Secret Scanning

<!-- tldr; Don’t rely only on SAST for secrets. Use a dedicated scanner at minimum, and if compliance/visibility matter, consider something like GitGuardian or or Trufflehog enterprise

I’ve been working in AppSec / secure software development for quite a while, and have had to set up secret scanning across different teams and stacks. Here’s how I’ve seen it play out in practice:

1. SAST vs. dedicated secret scanning

Even though some SAST tools advertise secret detection, they’re rarely as strong as dedicated tools. Secret scanning requires updated regexes, entropy checks, and context awareness — things SAST engines don’t focus on. In most setups, SAST is for code issues (injection, unsafe patterns, etc.), while secret scanning runs separately (pre-commit hooks, CI jobs, or continuous repo monitoring).

2. GitGuardian vs. Trufflehog (enterprise)

GitGuardian → polished SaaS, strong detection accuracy, dashboards, audit/compliance features, and easy integrations (Slack/Jira/SIEM). Great if you want visibility across the org.

Trufflehog → very flexible, great for scanning histories and many backends (git, S3, GCP, etc.), but you’ll need to invest more engineering effort if you want compliance/reporting at scale.Some teams actually pair them: Trufflehog for deep historical sweeps, GitGuardian for ongoing monitoring.

3. Alternatives

Gitleaks (what you’re already using) → solid for CI/CD pipelines but is not as polished as GG or TH for paid versions.

detect-secrets (Yelp) → good for pre-commit hooks, but not as actively maintained.

ggshield (GitGuardian CLI) → gives you both local and pipeline scanning tied to their SaaS backend.

Custom regex rules → handy for company-specific formats (internal API keys, JWTs, etc.).

Typical workflow

Pre-commit/pre-push: lightweight hooks (detect-secrets/gitleaks).

CI/CD: thorough scans (gitleaks/trufflehog).

Continuous monitoring: enterprise SaaS (GitGuardian). -->

> How can you ensure that sensitive information are not pushed to a repository?

This is one of the [OWASP Top Ten issues](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure) and
several bug bounties write-ups are related to this kind of issue, eg hard-coded credentials pushed by mistake.

You should scan your commits and your repository, and detect any sensitive information such as password, secret key, confidential, etc.
following the process shown in the picture.

The ideal approach is detecting and preventing the exposure of sensitive data before that they hit the repository,
because they are then visible in the history. In case of code hosting platforms, secrets can still linger
on the web and be searchable after you remove them from the repository.

A complimentary approach is scanning the repo for sensitive information, and then remove them;
note that when a credential is leaked, it is already compromised and should be invalidated.

## Detecting secrets in several locations

- **Detecting existing secrets** by searching in a repository for existing secrets.
- **Using Pre-commit hooks** in order to prevent secrets for entering our code base.
- **Detecting secrets in a pipeline** .

## Why Detecting Secrets?

- The secrets should not be hardcoded.
- The secrets should not be unencrypted.
- The secrets should not be stored in source code.
- The code history does not contain inadvertent secrets.

## Where and when to Detect Secrets?

![Pre Commit](/current-version/assets/images/pre-commit.png)

Well, the best location is the **pre-commit** location, This ensure that before a secret actually enters your code base, it is intercepted, and the developer or to committer gets a message. Using **SAST IDE plugins** comes in handy while trying to fix such issues where the IDE warnings can be found as soon as the developer writes an insecure code with security misconfigurations.  Another location is the build server or the **build** process. The build server retrieves source code, which is already committed and then it can analyze the source code where it contains new secrets or when it contains known secrets that the secrets are actually validated or audited.

---

Here are some helpful tools to automatically scan repositories for sensitive information.
Scans can be implemented directly in our pipeline, and be repeatable and efficient.

## Tools[^1]

### Open-source

- [Git Hound](https://github.com/ezekg/git-hound) - Git plugin that prevents sensitive data from being committed
- [git-secrets](https://github.com/awslabs/git-secrets) - Prevents you from committing secrets and credentials into git repositories
- [Github Secret Scanning](https://docs.github.com/en/code-security/secret-scanning) - Github built in feature for secret detection
- [gittyleaks](https://github.com/kootenpv/gittyleaks) - Find sensitive information for a git repo
- [Repo-supervisor](https://github.com/auth0/repo-supervisor) - Scan your code for security misconfiguration, search for passwords and secrets
- [truffleHog](https://github.com/dxa4481/truffleHog) - Searches through git repositories for high entropy strings and secrets, digging deep into commit history

### Commercial

- [BluBracket](https://blubracket.com) - Prevent secrets and credentials in code
- [GitGuardian](https://gitguardian.com) - Keep secrets out of your source code
- [GitHub Advanced Security](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning) - GitHub scans repositories for known types of secrets, to prevent fraudulent use of secrets that were committed accidentally
- [Nightfall](https://nightfall.ai) - Find and protect secrets and keys across the cloud
- [Spectralops](https://spectralops.io) - Developer-first Cloud Security
- [TruffleSecurity](https://trufflesecurity.com) - Unhearth your secrets

---

### Links

[^1]: Listed in alphabetical order.
