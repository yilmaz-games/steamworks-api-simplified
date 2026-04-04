# Contributing to steamworks-api-simplified

Thanks for wanting to help make Steam's API more accessible! Here's how you can contribute.

## Ways to Contribute

### Fix an error or outdated info
Steam updates their API occasionally. If you spot something wrong (a deprecated endpoint, incorrect parameter, misleading explanation), we want to know.

1. Fork this repo
2. Fix the issue in the relevant file
3. Submit a PR with a clear description of what was wrong and how you fixed it

### Add a missing endpoint
If you know about a Steam API endpoint that's not documented here:

1. Fork this repo
2. Add the endpoint in the appropriate section of `README.md`
3. Follow the existing format: endpoint URL, parameters table, response example, gotchas
4. Submit a PR

### Add a translation
We'd love translations in more languages! Current translations:

| Language | File | Status |
|----------|------|--------|
| English | [README.md](../README.md) | ✅ Complete |
| Turkish | [translations/README.tr.md](../translations/README.tr.md) | ✅ Complete |
| German | [translations/README.de.md](../translations/README.de.md) | ✅ Complete |

**To add a new language:**

1. Fork this repo
2. Copy `README.md` to `translations/README.{language-code}.md` (e.g., `README.fr.md`, `README.ja.md`, `README.ko.md`)
3. Translate all prose, headers, and table headers
4. Keep code blocks, URLs, API endpoints, and JSON examples in English
5. Add a link back to the English version at the top
6. Add links to other existing translations at the top
7. Submit a PR

Use [ISO 639-1 language codes](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) for the file suffix.

**To update an existing translation:**
If the English README has been updated and a translation is out of date, feel free to update it.

### Improve explanations
If something in the guide confused you, it probably confuses others too. PRs that make things clearer are always welcome, especially in the "New to APIs?" section and the "Common Gotchas" section.

## Guidelines

- **Keep it practical.** This is a guide for people who need to get things done, not an encyclopedia.
- **Follow the existing format.** Look at how other endpoints are documented and match the style.
- **Test your examples.** If you add a code sample or endpoint URL, make sure it actually works.
- **One change per PR.** Fixing a typo? Great. Adding an endpoint AND restructuring a section? Please split it into two PRs.
- **No promotional content.** This is a community resource. Don't add links to your products or services.

## Reporting Issues

Not sure how to fix something? Just open an issue:

- **[Report incorrect info](https://github.com/yilmaz-games/steamworks-api-simplified/issues/new?template=bug-report.yml)**: Something is wrong or outdated
- **[Suggest improvement](https://github.com/yilmaz-games/steamworks-api-simplified/issues/new?template=improvement.yml)**: An endpoint is missing or something could be explained better
- **[Volunteer for translation](https://github.com/yilmaz-games/steamworks-api-simplified/issues/new?template=translation.yml)**: Want to translate the guide into a new language

## Code of Conduct

Be helpful, be kind. This is a resource by indie developers for indie developers. We're all figuring things out together.

---

Thanks for making this guide better for everyone! 🙏
