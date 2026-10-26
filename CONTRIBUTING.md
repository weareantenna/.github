# Pull requests at Antenna

These rules apply to every repo in `weareantenna`. A repo with its own `CONTRIBUTING.md` or PR template overrides them.

## Branch and title

- Branch: `<type>/<ticket>-<short-slug>`, e.g. `fix/417-klantreferentie-optioneel`. Types: `feat`, `fix`, `chore`, `hotfix`, `release`.
- Title: what the PR does, readable without the ticket. Start with the ticket number when there is one.
- One ticket per PR. Split unrelated changes.

## Description

Use the PR template and fill in every section. Write "n/a" with a reason instead of deleting a section.

In "Merge danger", say whether a revert fully undoes the change (two-way door) or not (one-way door), and how much breaks if it goes wrong. A one-way door asks for a closer review.

## Screenshots

Required when anything visible changed: before and after, at the size a user sees it.

- In the browser: drag the images into the description.
- From the command line or an agent, without putting images in the repo history: upload them to the `pr-assets` branch and paste the printed Markdown into the description.

  ```sh
  gh api repos/weareantenna/.github/contents/scripts/pr-screenshots.sh --jq .content | base64 -d > /tmp/pr-screenshots.sh
  sh /tmp/pr-screenshots.sh <pr-number> before.png after.png
  ```

## Before you open the PR

- Run the repo's lint and tests, and say in "How it was tested" what you ran.
- Mention migrations, new environment variables and config changes in the description.
