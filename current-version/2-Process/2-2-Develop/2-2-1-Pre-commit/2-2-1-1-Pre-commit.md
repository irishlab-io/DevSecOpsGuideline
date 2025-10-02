# Pre-Commit

> Prevention is better than cure
> It is better to stop something bad from happening than it is to deal with it after it has happened.

The phrase is attributed to [Benjamin Franklin](https://en.wikipedia.org/wiki/Benjamin_Franklin), who famously advised the citizens of Philadelphia in the 18th century that "an ounce of prevention is worth a pound of cure," relating it to preventing fires.

The `pre-commit` phase is important because it can prevent security issues before they are ever recoded as part of the git history.

## Git hooks

Like many other version control (also known as revision control, source control, and source code management), Git has a way to fire off custom scripts when certain important actions occurs.  These scripts are known as `git hooks` that are self-executed before or after events like committing, pushing, or receiving. They are categorized into client-side and server-side hooks.

![Pre Commit](/current-version/assets/images/git_hook_workflow.png)

### Server-Side Hooks

Server-side hooks resides in server-side repositories (i.e.: a central repository, or a developer’s public repository). When attached to the official repository, some of these can serve as a way to enforce policy by rejecting certain commits.

You should keep in mind that server-side hooks can become very ressources intensive and cause significant impact.  For these reasons, server-side hooks are rarely use and often prohibited by some vendors.  Yet these exists and the best exemple for such capability is most likely [GitHub: Push Protection](https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection), an implementation of a pre-receive hooks used as part of GitHub secret scanning feature.  This allows the VCS server to reject a `git push` preventing an end user to accidentally leak a secret publicly.

<details>
<summary>List of server-side hooks</summary>

- post-receive: Executed after a successful push, after all refs have been updated. Used for notifications, triggering CI/CD pipelines, etc.
- post-update: Similar to post-receive, but invoked with a list of updated refs.
- pre-receive: Executed when a push is received by the server, before any refs are updated. Used for enforcing policies.
- reference-transaction: Executed during a reference transaction, allowing validation or actions related to ref updates.
- update: Executed once per ref being updated during a push. Allows fine-grained control over which refs are accepted.

</details>

### Client-Side Hooks

Client-side hooks are custom scripts that are triggered on a developer's local machine at the various trigger point in the git workflow lifecycle.

They can be found within the `.git/hooks` directory of a local repository and are not tracked by the git repository.  By default, Git provides examples of how you can use a varities of hooks. Theses exemples are suffix as  `<client-side hooks>.sample` preventing them from being run.  Removing the suffix will trigger the associated hook automatically during such git event occurences.

Git hooks script can access system wide languages and tool although low level programming langauge such as `bash`, `python`, `perl` are typically used.  Given these hooks are not included as part of the repository, `native git hooks` can be cumbersome to share and maintain at scale within an organization.  Each developer needs to set up these hooks individually for each repository or using advanced-global git configuration making these unuseful to manage them across a team.

<details>
<summary>List of client-side hooks</summary>

- applypatch-msg: Invoked by git am to process the commit message of a patch.
- commit-msg: Executed after the user has entered a commit message. Used for validating the message format.
- post-applypatch: Invoked by git am after applying a patch.
- post-checkout: Executed after a git checkout or git switch command has updated the working tree.
- post-commit: Executed after a commit is successfully created.
- post-index-change: Executed after the index is changed by commands like git update-index or git add.
- post-merge: Executed after a successful git merge command.
- post-rewrite: Executed after commands that rewrite history (e.g., git commit --amend, git rebase).
- pre-applypatch: Invoked by git am before applying a patch.
- pre-auto-gc: Executed before Git automatically runs garbage collection.
- pre-commit: Executed before a commit is created. Used for validating code style, running tests, etc.
- pre-merge-commit: Similar to pre-commit, but specifically for merge commits.
- pre-push: Executed before git push attempts to update remote refs.
- pre-rebase: Executed before a rebase operation begins.
- prepare-commit-msg: Executed after the default commit message is created, but before the editor is launched. Allows modification of the message.
- push-to-checkout: Executed on the client side when a git push is performed and the remote repository is configured for push-to-checkout.
- sendemail-validate: Used by git send-email to validate email messages.

</details>

## Usages

One of the key objectives of the DevSecOps  mindset is to shift-left security and checks earlier in the process.  This reduce the amount of churns therefore increasing quality, reducing risk and improving time to resolution.  Although, it is important to recognized that adding these early chcek requirement increase the already heavy mental load for the developpers whom are already joggling with several complexe topics.  Therefore automation and simplicity of usage is key and given the inconvenience of `native git hooks`, several open-source [tool](#tools1) are attempting to simplify such management, execution and distribution.

Two of the most common `git hooks` are the `pre-commit` and `commit-msg` given their position within the git workflow.

### `pre-commit` hooks

The `pre-commit` hooks is trigger between the `git add` and `git commit` operation (also known as the staging area) there it is the perfect timing to validate any new code changes prior to including these within the git history.  Without getting into to much details about the git object model (blob, tree, commit, tag, etc...), the nature of the git makes every commits perpetual meaning that every version of the code is tracked under the hood.  Therefore accidentally adding and committing sensitive informations and quickly removing these will retain such sensitive informations as part as the git history.  Although not obvious for newcmers, it is fairly easy to extract historic commit and therefore access any sensitive information.  Therefore `pre-commit` hooks triggered and validating the code content prior to code inclusion solve this problem because this hook execution failure will result in state reset and not commit will be added preserving the precious git history.

Typical `pre-commit` hooks will validate for:

- [Secret Scanning](../2-2-1-Pre-commit/2-2-1-2-Secrets-Management.md): Most likely the most useful security related hooks.
- [Linting](../2-2-1-Pre-commit/2-2-1-3-Linting-code.md): Most likely the most common usage for `pre-commit` hooks.
- Format: Standardize common format based on define rulesets.
- Security tools: Several security tools can be run locally capturing issue earlier.
- Misconfiguration: Common misconfiguration can be catch and avoid developpers annoyances.
- Testing: Running series of local unit test and code coverage

### `commit-msg` hooks

The `commit-msg` hooks is trigger after the `prepare-commit-msg` operation has completed but prior to the `post-commit`.  Typically this is the last moment before the developper execute the `git push` action promotion the commits from it's local repository to the remote repository.  Given the commit is completed including the commit message; it is possible to validate the entirety of the upcoming code push.  This is the last opportunity to capture any issue ahead of these commits to be made **public**.  The word **public** have no relationship with the git repository access control model but rather public amongst other whom have already repository accesses.

- Message standards: Enforce specific standard for the commits messages

### Others hooks

Almost any of the standard [Client-Side Hooks](#client-side-hooks) can be used by the various framework automating `git hooks` execution.  Depending on your workflow he `pre-applypatch` and `post-applypatch` might be useful to you but its usage are less common.

## Skipping

Programming convention can generation conflictual and confrontational situatiosn where not all team members agrees on some of them.  The usage of `pre-commit` hooks attempts to fosther a shift-left mindset.  Automating code linting and formating can reduce the amount of churn experienced during the code review process where personal **nit-pick** can overextend unnecessarly.  Given the various convention are defined as code via various configuration files used by the `pre-commit hooks` its become harder to disagree with these rules on pure personnal preference and the enterity of the codebase increase in cohesion,

This being said, sometimes because of problematic or time sensitive situation or simply due to personal choicea developper might be required to commit out of spec code.  Therefore it can be common for developpers to sovereignly decide to skip some or all pre-commit validation with appending the `--no-verify` flag to its `git commit` operations.  This git native funtionnality will **skip all installed hooks**.  Depending on the severity of the rulesets being skipped some downstream processes might capture such omission  (i.e.: code review, CI pipeline, etc...).

- This delibarate slippage may be a well taugh out process of acquiring **technical debt** in order to increase velocity such as not achieving the target code coverage expected by this project temporarly to deliver a new feature.
- Another reason some developpers might skip some `pre-commit` hooks is due to false positive finding that can be explain as part as the code review phase.  Which might lead to `pre-commit` hooks rulesets modifications.
- Finally a disgruntled team members might simply reject the usage of such rulesets and mechanism constraining their workflow.

The project leadership should be included in all slippage and consider the fundamental root cause of these.

- **Technical debt** can be beneficial but must be monitored
- False-positive are a common thing but must be investigate
- Resentful developpers are not constrain to use `pre-commit` hooks but must comply with code format, quality and security expectation; which `pre-commit` hooks can assist automating.
- Lastly, it is the project leadership prerogative to impose coding standard it desire.

## CI Pipeline

CI pipeline should follow the [Verification and Validation](https://en.wikipedia.org/wiki/Verification_and_validation) (also known as V&V) quality assurance processes.  This implies that all verification made using `pre-commit` hook should be ran in some shape or form during the integration phase.  It can be convenient to **re-run** the [pre-commit frameworks](#tools1) as the first integration steps using the repository configuration.  This will ensure that if some project members have committed code that does not meet the expect code quality or security, a **failed-fast** will occurs quikcly capturing the issue.  Such failure should prevent any further activities part of the code integration process and prevent the project member contribution to be halted until correction are made.

This is also an ideal means to catch disgruntled team members that are not following the agreed coding convention.  A common reaction should be to put on hiatus any code review by peer until the `pre-commit` hook run pass successfully showcasing it meet the expected quality and security.

Finally some `pre-commit` hook might be disabled from this initial **re-run** check in favor of a further **full-fat** scanner integration.  Given `pre-commit` hook are kept lightweight for ease of usage they might not perform all check and validation that a dedicated tool such as secret scanner, sast, sca and others might do.  Also these dedicate tools might performs more in depth valiation that goes beyond what the [pre-commit frameworks](#tools1) allows to achieve.

In the end, parity between `pre-commit` hook and CI pipeline can not always be achieve but it is ideal to converge as closely as possible to reduce churn.

## Considerations

Here are some consideration to ponder upon when using `pre-commit` hooks within a project.

### Improper installation

The value factor using [pre-commit frameworks](#tools1) lives in the portability and distribution of common sets of rules which can be difficult using `native git hooks`.  Although upon a project `git clone` most tools are not self-bootstrapping on the developper workstation.  The various framework requires initial setup actions to be manually performed by the developper which can easily be forgotten leading to some early contribution commits not following the agreed upon rulesets.

This can be mitigate with proper project on-boarding and various checklist validation during the code merging process.

### Annoyances

Keeping `pre-commit` hooks should be as an elective toolchain that developpers choose to opt-in can reduce the annoyance factor for some developpers.  The goal of `pre-commit` hooks is to offload the mental burden of running common useful operation prior to code commits in order to achive higher quality and security.  Although, some developpers might already achieve this without any automation after years of practices.  As long as, the code quality and security is achieved, the developpers ar free to decide how they achieved it.

It should be clear that the project expectation is to achieve the quality and security goals; not running the various `pre-commit` hooks.

### Performance

Given that `pre-commit` hooks are executed during the various and common `git` operations they should be kept to be extemely minimalistic.  Long running tests suites or security tool that requires several minutes to complete well break a developper natural pace and workflow therefore reducing such capabilities feature rate.  Executing `pre-commit` hooks are not **free** but should not require more than a few seconds to complete especially that [several frameworks](#tools1) leverage multi-threading and modern hardware capabilities.

This should be monitored and long running task move toward the CI pipeline focusing on value added actions.

### Context switching

Capturing issues and problems early means they are faster and cheaper to solve.  But by definition, they are slowing down the developper natural workflow and adding more responsabilitis to understand why such and such issues are meant, especially in regards with security scans.  This constant capture of new issues can create a significant impact on the developper continuously context switch between completing the task at hand and addressing security or code quality requirement.

Reduce the scope of the mandatory `pre-commit` hooks towards they main goals.

### Velocity

During new features developement, it can be common not to meet the various `pre-commit` hooks expectation.  Adding large new features without proper unit test, including hardcode value or improper code formatting are all typically expected by the natural developpers workflows.  In order to maintain his target pace, a developper might elect to disable `pre-commit` hooks as a whole for an extended duration creating a series of unvalidated commits.  Such strategy is acceptable as long as the codebase including the git history are updated to comply with the expected code quality and security.

Enabling the `pre-commit` hooks and usage of `git rebase` operation to rewrite the git history prior to pushing the code is acceptable.

### Initial on-boarding

Legacy project codebase comes with historic **technical debt** in the shape of much discrepency with the target code standard, quality and security.  On-boarding a legacy project to use `pre-commit` hooks can be difficult given the initial correction to ensure a minimal adoption of the expected rulesets.  It is to be expect the initial momentum required to make all required correction and achieve the project adoption via possiblity a Pull Request might take several days or weeks.  This is mostly due that typically code linting and formatting will introduce a **large number of Line Of Code** changes which depending on the project maturity level and test coverage scares the various contributors that some unexpected breakage might occurs.

Introducing a minimal scope of the mandatory `pre-commit` hooks and including optional hooks to be run ad-hoc will ease this initial transition.

---

## Tools[^1]

### Open-source

- [CaptainHook](https://github.com/captainhook-git/captainhook) - Git hooks manager for PHP developers
- [Git Build Hook Maven Plugin](https://github.com/rudikershaw/git-build-hook) - Install Git hooks and config during a Maven build
- [Husky](https://typicode.github.io/husky/) - Git hooks made easy 🐶 woof!
- [Lefthook](https://github.com/evilmartians/lefthook) - Fast and powerful Git hooks manager for any type of projects
- [Overcommit](https://github.com/sds/overcommit) - A fully configurable and extendable git hook manager
- [Pre-Commit](https://pre-commit.com/) - A framework for managing and maintaining multi-language pre-commit hooks
- [Simple-git-hooks](https://github.com/toplenboren/simple-git-hooks) - A simple git hooks manager for small projects

### Links

- [Atlassian docs](https://www.atlassian.com/git/tutorials/git-hooks)
- [awesome-git-hooks](https://github.com/CompSciLauren/awesome-git-hooks)
- [Git Hooks](https://git-scm.com/docs/githooks)
- [Kinksta Blog](https://kinsta.com/blog/git-hooks/)
- [RedHat Blog](https://www.redhat.com/en/blog/git-hooks)

[^1]: Listed in alphabetical order.
