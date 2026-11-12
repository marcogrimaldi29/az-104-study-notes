# Contributing to AZ-104 Study Notes

Thanks for helping keep these notes accurate. They cover **Microsoft Exam AZ-104: Microsoft Azure Administrator**, and Azure changes fast: services get renamed, SKUs are retired, and defaults and limits move between one revision of the study guide and the next. Reports from people studying with the notes are the best way to catch that.

> [!IMPORTANT]
> **This repo accepts issues, not pull requests.** Pull requests are turned off: I write every change myself and check it against the official documentation, so the notes keep one voice and stay consistent from page to page. If you know exactly how something should read, put your proposed wording in the issue. It's just as useful.

---

## Contents

- [Ways to Help](#-ways-to-help)
- [Before You Open an Issue](#-before-you-open-an-issue)
- [Opening an Issue](#-opening-an-issue)
- [Out of Scope](#-out-of-scope)
- [What Happens Next](#-what-happens-next)
- [Code of Conduct](#-code-of-conduct)
- [Security](#-security)

---

## 🙌 Ways to Help

Every report fits one of the five types in the issue template:

| Type | What it covers | For example |
|------|----------------|-------------|
| 🐞 **Bug** | A broken link, a diagram that doesn't render, or search or navigation not working | A link returns 404, a Mermaid diagram shows up as plain code, or search finds nothing for a term that's on the page |
| ✏️ **Content error** | Something in the notes is wrong or misleading | An Azure CLI or PowerShell command uses the wrong parameter, an SLA percentage is wrong, or a table puts a feature in the wrong SKU tier |
| 🕰️ **Outdated content** | Azure or the exam has changed since the page was written | A service or SKU was renamed or retired, a default or limit has changed, or the study guide was revised after October 26, 2023 |
| 📱 **Device issue** | The site looks or behaves wrongly on a specific device or browser | A wide comparison table is cut off on a phone in portrait, or the navigation menu won't close on a tablet |
| 💬 **Other** | Anything else, including suggestions | A topic that would read better as a decision table or a diagram, or an exam caveat worth adding |

---

## 🔎 Before You Open an Issue

- **Search the existing issues**, [open and closed](https://github.com/marcogrimaldi29/az-104-study-notes/issues?q=is%3Aissue). If yours is already there, add a 👍 reaction or a comment with anything new rather than opening a duplicate.
- **Check the date.** The notes are written against the **skills measured as of October 26, 2023**. If the "Skills measured as of" heading in the [study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104) shows a newer date, say so in the issue — it tells me which revision to compare against.
- **Check the current docs.** A quick look at the [Azure documentation](https://learn.microsoft.com/en-us/azure/), or the [Microsoft Entra documentation](https://learn.microsoft.com/en-us/entra/) for identity topics, usually confirms whether something has changed, and gives you the link the issue asks for.
- **One problem per issue.** Separate issues are easier to track, fix and close. Several typos on the same page can go in one issue.
- **For a site bug, hard-refresh first** (Ctrl+F5, or Cmd+Shift+R on a Mac) in case your browser is showing an older copy of the page.

---

## 📝 Opening an Issue

1. Open a new issue with the [🐞 Report a problem with the notes](https://github.com/marcogrimaldi29/az-104-study-notes/issues/new?template=ISSUE_TEMPLATE.md) template.
2. Give it a title that names the page and the problem, for example *"04 — Virtual Networking: VNet peering section says peering is transitive"* rather than *"Mistake on page"*.
3. Fill in the template's sections. Leave out any that don't apply.

> [!TIP]
> To link straight to a section, click it in the page's **Table of Contents**, or hover over its heading and click the link icon next to it, then copy the address from your browser.

### What Makes a Report Easy to Fix

- **A link to the exact section**, not just the page
- **The text as the notes have it**, quoted, so it can be found even if the page has moved on
- **An official source** for any content change — Microsoft Learn is best. A report without one may take longer, because every change has to be checked against the documentation before it goes in
- **For a command that doesn't work as written:** the error message, and the Azure CLI or Az PowerShell version you ran it with
- **For bugs and device issues:** the steps to reproduce it, what you expected and what happened instead, the device, operating system and browser, and a screenshot if it shows the problem
- **Your proposed wording**, if you have it. With pull requests turned off, this is the closest thing to suggesting an edit directly

---

## 🚫 Out of Scope

To keep the notes accurate and the issue tracker useful, please don't open issues for:

- **Live exam content.** Questions, answers or topics remembered from a real exam break [Microsoft's exam security policy](https://learn.microsoft.com/en-us/credentials/support/exam-and-assessment-lab-security-policies) and the [exam candidate agreement](https://learn.microsoft.com/en-us/credentials/support/certification-exam-candidate-agreement) every candidate accepts. Issues or comments that include them will be closed and the content removed.
- **Exam booking, scores or retakes.** [Microsoft Credentials support](https://learn.microsoft.com/en-us/credentials/support/help) can help with those.
- **Help with your own Azure subscription or tenant.** For troubleshooting a deployment, a VM, a network or a policy assignment of your own, ask on [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/) or open an Azure support request.
- **Anything private.** Issues are public, so leave out subscription and tenant IDs, account keys, SAS tokens, connection strings, public IP addresses and customer details, including in screenshots.
- **Unofficial sources.** Exam dump sites, third-party practice tests and AI chat answers aren't accepted as the source for a content change.
- **Promotion.** Links to courses, products or services, affiliate links and paid placements won't be added.
- **Security vulnerabilities.** Please [report these privately](https://github.com/marcogrimaldi29/az-104-study-notes/security/advisories/new), as described in the [security policy](SECURITY.md), not in a public issue.

---

## 🔄 What Happens Next

- I maintain the notes on my own, in my spare time, so replies are best effort and there's no fixed response time.
- I check each report against the official documentation. If something needs clarifying, I'll ask on the issue, so keep an eye on your GitHub notifications.
- When a fix goes in, the issue is closed with a note or a link to the change. The live site updates shortly after.
- If nothing changes, the issue is closed with the reason, such as out of scope, can't be verified, or working as intended.
- An issue waiting on a reply may be closed after a while. Comment with the missing details and it can be reopened.

---

## 🤝 Code of Conduct

By opening, commenting on or reacting to issues here, you agree to follow the [Code of Conduct](CODE_OF_CONDUCT.md). Comments that are abusive, spam or share live exam content may be hidden, edited or locked.

---

## 🔒 Security

Found a security problem with the site or this repo? Please don't report it in a public issue. Use [Report a vulnerability](https://github.com/marcogrimaldi29/az-104-study-notes/security/advisories/new) instead, so only you and I can see it. The [security policy](SECURITY.md) has the details.

---

If the notes helped you, the README's [💚 Support](https://github.com/marcogrimaldi29/az-104-study-notes#-support) section has a couple of ways to show it. Thanks for helping make them better for the next AZ-104 candidate.
