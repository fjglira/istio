# Istio AI Policy

Istio is a project that welcomes any contributions, including those that are assisted by AI tools. Contributors to the project are able to use AI tools to assist in their work, but they remain fully responsible for the content they submit.

This policy shows how the Istio project expects contributors to use AI tools responsibly. It outlines the principles, requirements, and best practices for AI-assisted contributions.

The most relevant points from this policy are:
* AI use should be disclosed when it substantially contributes to the content being submitted. We expect contributors to clearly indicate AI assistance in their submissions and we are fine with such contributions as long as disclosure is made.
* Using a AI tool to answer question on issues, PR's or reviews is not allowed in this project. We expect to have conversation or discussion with other humans directly.
* Large AI generated contributions are not allowed. We expect contributor to be able to split work into smaller, human reviewed pieces. This will ensure that maintainers can effectively review and validate the content.

For a first contribution, choose one of two paths: make a simple change to get started, or, for a complex use case, open an issue and discuss the topic with the community before submitting a PR. Link the issue in your PR, and bring the topic to a relevant [Working Group meeting](https://istio.io/latest/about/community/working-groups/) for discussion. You don't need to be an expert if you're addressing a specific use case, but you should understand the topic well enough to explain it and take part in the discussion.

As part of the CNCF, Istio project is also bound by the [Linux Foundation's policy on AI and copyright](https://www.linuxfoundation.org/legal/generative-ai), which can be broadly summarized for the purpose of this document as:

* Copyright for all submissions is assigned by the submitter to the project. AI tools cannot hold copyright.

And we think it's important to make this explicit for contributors: while AI tools can assist in creating content, the human contributor retains full responsibility and copyright for their submissions.

The consequences of violating this policy are described in [AI Policy Violation and Unacceptable Use of AI](#ai-policy-violation-and-unacceptable-use-of-ai).

## Purpose and Scope

The main goal of this policy is to establish a clear framework for the responsible use of AI tools in contributions to the Istio project. Also, it aims to ensure that all contributions made are properly reviewed and validated by human contributors.

Any AI generated content that has not been carefully reviewed can consume significant maintainer time and make reviews less efficient and reliable. Contributors are responsible for verifying the accuracy, quality, and suitability of all submitted work. Maintainer review is an additional safeguard, not a substitute for that responsibility.

This policy applies to all contributions to the Istio project and it's not only scoped to code, but also includes tests, documentation, release notes, issues, PRs, and review comments across all `istio/*` repositories.

## Definitions

To set a baseline, let's define some key terms related to the AI use:

* **AI tools:** Software or agents that assist contributors in creating content, including code, text, or other forms of contribution.
* **Assistance:** Help from AI tools, such as autocomplete suggestions, grammar fixes, or recommendations. Assistance may also include code or text generation; whether that content is substantial depends on how much of it is included in a contribution.
* **Generated content or substantial AI contribution:** Content produced by an AI tool and intended for inclusion in a contribution. A contribution is considered substantially AI-generated when an AI tool produces most or all of it.

Based on these definitions the contributors are going to be responsible for distinguishing between what is assistance and what is a substantial AI-generated contribution. They must ensure that any substantial AI-generated content is properly reviewed, verified, and disclosed according to this policy.

## Core Principles

The Istio project expects AI tools to be used responsibly in contributions. These principles guide that use:

* Human contributors are accountable for everything they submit, including AI-assisted or AI-generated content.
* Contributors must review and verify AI-generated content and be able to explain the changes they submit.
* Contributors should distinguish ordinary assistance from substantial AI-generated content and disclose substantial use as described in this policy.
* AI tools must not replace direct human participation in issue, pull request, or review discussions, as described in [Issues, PRs, Reviews and any kind of Contribution](#issues-prs-reviews-and-any-kind-of-contribution).
* AI tools must not impersonate the contributor. Comments, replies, descriptions, and any other project communication must be written and answered by a human, and AI bots or automated systems must not post on a contributor's behalf. See [AI Policy Violation and Unacceptable Use of AI](#ai-policy-violation-and-unacceptable-use-of-ai).
* Contributions should meet the project's standards regardless of how they were produced. Unreviewed or low quality contributions may be rejected; see [AI Policy Violation and Unacceptable Use of AI](#ai-policy-violation-and-unacceptable-use-of-ai).


## Contributor Requirements

We expect to any contributor to follow the Core Principles outlined above when using AI tools in their contributions. This includes reviewing and verifying AI-generated content, distinguishing between ordinary assistance and substantial AI-generated contributions, and disclosing substantial use as required by this policy.

## Disclosure

Please add a disclosure in the PR description or other appropriate location when you have used AI tools to generate substantial content for the contribution. It should be something like this:

```
AI disclosure: Used AI tools to generate substantial content.

AI disclosure: Used AI tools to generate substantial content. Reviewed and verified by human contributor.
```

We expect contributors to provide such disclosure. If the content of the contribution was created or assisted by AI tools but the human review and process was thorough and verified, it should be indicated as such in the disclosure, that will give the reviewers confidence that the contribution meets the project's standards. Be transparent about the role of AI in your contribution is the best practice.

## Issues, PRs, Reviews and any kind of Contribution

AI tools may assist with Istio contributions by helping contributors understand the project, brainstorm ideas, analyze submissions, summarize verified information, draft code or specifications, and edit text. These uses are acceptable as long as the contributor reviews and verifies AI-generated content before submission. Design documents and other substantial design content require a high level of human involvement and careful review by contributors, maintainers, and reviewers.

Contributors must review and verify AI-generated content, understand and be able to explain their changes, and take responsibility for what they submit. 

If AI tools generated substantial content for a contribution, disclose that use in the PR description, as explained in the [Disclosure](#disclosure) section. If there is no PR, include the disclosure in an appropriate location with the contribution. Contributors may also use labeling based on the [AIL framework](https://danielmiessler.com/blog/ai-influence-level-ail) (for example, `AIL-2`) to indicate AI's influence in their contributions.


## AI Policy Violation and Unacceptable Use of AI

Maintainers or project leaders may ask contributors whether they used AI tools. Contributors are expected to answer honestly to preserve trust and the integrity of the collaborative process.

Please don't publicly accuse someone of using AI based on writing style or other factors. Share concerns privately with maintainers, and follow the [CNCF Code of Conduct](https://www.cncf.io/conduct/). A single mistake won't lead to serious consequences, but repeated or intentional misrepresentation will be taken seriously.

Violations of this policy may result in consequences such as:

* Closing PRs or issues
* Banning contributors from participating in the project.

The following are examples of unacceptable use and policy violations:
* Using AI or automated systems to impersonate a contributor or post on their behalf. Project communications must come from a human.
* Misrepresenting AI-generated content as entirely human-created.
* Using AI to bypass human review and verification, or repeatedly submitting inaccurate or incomplete AI-generated content without proper review.
* Submitting content without the proper human review and verification.
* Using AI tools to generate content that violates the Istio Code of Conduct or other community guidelines.
* Only rely on AI tools to problem solving, architecture design and other technical decisions as an aid, not as a replacement for human judgment and review.

We from the Istio project care about the community and his contributions and as everyone is evolving with the use of AI we are evolving as well. We encourage a responsible and transparent use of AI tools, always ensuring that human review and verification is always part of the process.
