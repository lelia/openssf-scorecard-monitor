# OpenSSF Scorecard Monitor

**Simplify OpenSSF Scorecard tracking in your organization with automated markdown and JSON reports, plus optional GitHub issue alerts.**

*This project is part of [OpenSSF Scorecard](https://github.com/ossf/scorecard). Read [the announcement](https://github.com/ossf/scorecard-monitor/issues/79) for more details.*

## 🔮 About

If you're feeling overwhelmed by an avalanche of scorecards across your organizations, you can breathe easy: automation is here to make your life easier! Scorecard Monitor streamlines the process of keeping track of them all by providing a comprehensive report in Markdown and a local database in JSON with all the scores. To stay on top of any changes in the scores, you can also choose to get notifications through Github Issues.

## ✅ Requirements

Please ensure that any repository you wish to track with Scorecard Monitor has already been analyzed by [OpenSSF Scorecard](https://github.com/ossf/scorecard) at least once. This can be accomplished using the official [GitHub Action](https://github.com/ossf/scorecard-action) or the [Scorecard CLI](https://github.com/ossf/scorecard?tab=readme-ov-file#scorecard-command-line-interface).

It's also possible that some repositories in your organization are already being [automatically tracked](https://github.com/ossf/scorecard/blob/main/docs/faq.md#can-i-preview-my-projects-score) by OpenSSF in this [CSV file](https://github.com/ossf/scorecard/blob/main/cron/internal/data/projects.csv) via weekly cronjob. One caveat: Automatically tracked projects _do not_ include [certain checks](https://github.com/ossf/scorecard/issues/3438) in their analysis (`CI-Tests,Contributors,Dependency-Update-Tool,Webhooks`).

If you're not sure whether a specific project is already using OpenSSF Scorecard, you can always spot-check with the following URL pattern: `https://securityscorecards.dev/viewer/?uri=github.com/<ORG_NAME>/<REPO_NAME>` (substitute `<ORG_NAME>` and `<REPO_NAME>` as appropriate). The [Scorecard API](https://api.scorecard.dev/) is also able to fetch scores for a given repository.

## 📺 Tutorial

This section is coming soon.
If you would like to contribute to the documentation, please feel free to open a pull request for review.

## ❤️ Awesome Features

- Easy to use with great customization
- Easy to patch the scoring as the reports includes a direct link to [StepSecurity](https://app.stepsecurity.io)
- Easy way to visualize results with [Scorecard Visualizer](https://ossf.github.io/scorecard-visualizer/#/projects/github.com/nodejs/node) or [deps.dev](https://deps.dev/project/github/nodejs%2Fnode)
- Cutting-edge feature that effortlessly compares OpenSSF scorecards between previous and current commits with [The Scorecard Visualizer Comparator](https://ossf.github.io/scorecard-visualizer/#/projects/github.com/nodejs/node/compare/39a08ee8b8d3818677eb823cb566f36b1b1c4671/19fa9f1bc47b0666be0747583bea8cb3d8ad5eb1)
- Discovery mode: list all the repos in one or many organizations that are already being tracked with [OpenSSF Scorecard](https://github.com/ossf/scorecard)
- Reporting in Markdown with essential information (hash, date, score) and comparative against the prior score
- Self-hosted: The reporting data is stored in JSON format (including previous records) in the repo itself
- Generate an issue (assignation, labels..) with the last changes in the scores, including links to the full report
- Automatically create a pull request for repositories that have branch protection enabled
- Easy to exclude/include new repositories in the scope from any GitHub organization
- Extend the markdown template with you own content by using tags
- Easy to modify the files and ensure the integrity with JSON Schemas
- The report data is exported as an output and can be used in the pipeline
- Great test coverage

### 🎉 Demo

#### Sample Report

![sample report](.github/img/report.png)

_[Sample report](https://github.com/nodejs/security-wg/blob/main/tools/ossf_scorecard/report.md)_

#### Sample Issue

![sample issue preview](.github/img/issue.png)

_[Sample issue](https://github.com/nodejs/security-wg/issues/885)_

## :shipit: Used By

- [Nodejs](https://github.com/nodejs): The Node.js Ecosystem Security Working Group is using [this pipeline](https://github.com/nodejs/security-wg/blob/main/.github/workflows/ossf-scorecard-reporting.yml) to generate a [report](https://github.com/nodejs/security-wg/blob/main/tools/ossf_scorecard/report.md) with scores for all the repositories in the Node.js org.
- [One Beyond](https://github.com/onebeyond): The Maintainers are using [this pipeline](https://github.com/onebeyond/maintainers/blob/main/.github/workflows/security-scoring.yml) to generate a scoring report inside [a specific document][onebeyond-report], in order to generate a [web version](https://onebeyond-maintainers.netlify.app/reporting/ossf-scorecard) of it
- [NodeSecure](https://github.com/NodeSecure): The Maintainers are using [this pipeline](https://github.com/NodeSecure/Governance/blob/main/.github/workflows/ossf-scorecard-reporting.yml) to generate a scoring report and notification issues.
- **[More users](https://github.com/ossf/scorecard-monitor/network/dependents)**

[onebeyond-report]: https://github.com/onebeyond/maintainers/blob/main/docs/06-reporting/01-scorecard.md

## 📡 Usage

### Standalone with auto-discovery version

With the following workflow, you will get the most out of this action:

- Trigger manually or by Cron job every Sunday
- It will scan the org(s) in scope looking for repositories that are available in the OpenSSF Scorecard
- It will store the database and the scope files in the repo
- It will generate an issue if there are changes in the score

```yml
name: "OpenSSF Scoring"
on: 
  # Scheduled trigger
  schedule:
    # Run every Sunday at 00:00
    - cron: "0 0 * * 0"
  # Manual trigger
  workflow_dispatch:

permissions:
  # Write access in order to update the local files with the reports
  contents: write
  # Write access only required if creating PRs (see Advanced Tips below)
  pull-requests: none 
  # Write access in order to create issues
  issues: write
  packages: none

jobs:
  security-scoring:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: OpenSSF Scorecard Monitor
        uses: ossf/scorecard-monitor@v2.0.0-beta8
        with:
          scope: reporting/scope.json
          database: reporting/database.json
          report: reporting/openssf-scorecard-report.md
          auto-commit: true
          auto-push: true
          generate-issue: true
          # The token is needed to create issues, discovery mode and pushing changes in files
          github-token: ${{ secrets.GITHUB_TOKEN }}
          discovery-enabled: true
          # As an example nodejs Org and Myself
          discovery-orgs: 'UlisesGascon,nodejs'
```

### Options

- `scope`: Defines the path to the file where the scope is defined
- `database`: Defines the path to the JSON file usage to store the scores and compare
- `report`: Defines the path where the markdown report will be added/updated
- `auto-commit`: Commits the changes in the `database` and `report` files
- `auto-push`: Pushes the code changes to the branch
- `generate-issue`: Creates an issue with the scores that had been updated
- `issue-title`: Defines the issue title
- `issue-assignees`: List of assignees for the issue
- `issue-labels`: List of labels for the issue
- `github-token`: The token usage to create the issue and push the code
- `max-request-in-parallel`: Defines the total HTTP Request that can be done in parallel
- `discovery-enabled`: Defined if the discovery is enabled
- `discovery-orgs`: List of organizations to be includes in the discovery, example: `discovery-orgs: owasp,nodejs`. The OpenSSF Scorecard API is case sensitive, please use the same organization name as in the GitHub url, like: <https://github.com/NodeSecure> is `NodeSecure` and not `nodesecure`. [See example](https://github.com/NodeSecure/Governance/issues/21#issuecomment-1474770986)
- `report-tags-enabled`: Defines if the markdown report must be created/updated around tags by default is disabled. This is useful if the report is going to be include in a file that has other content on it, like docusaurus docs site or similar
- `report-start-tag` Defines the start tag, default `<!-- OPENSSF-SCORECARD-MONITOR:START -->`
- `report-end-tag`: Defines the closing tag, default `<!-- OPENSSF-SCORECARD-MONITOR:END -->`
- `render-badge`: Defines if the OpenSSF Scorecard badge must be rendered in the reporter to only show the score
- `report-tool`: Defines the reporting review tool in place: `scorecard-visualizer` [Example](https://ossf.github.io/scorecard-visualizer/#/projects/github.com/nodejs/node) or `deps.dev` [Example](https://deps.dev/project/github/nodejs%2Fnode), by default `scorecard-visualizer`

### Outputs

- `scores`: Score data in JSON format

```yml
name: "OpenSSF Scoring"
on: 
  # ...

permissions:
  # ...

jobs:
  security-scoring:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: OpenSSF Scorecard Monitor
        uses: ossf/scorecard-monitor@v2.0.0-beta8
        id: openssf-scorecard-monitor
        with:
          # ....
      - name: Print the scores
        run: |
          echo '${{ steps.openssf-scorecard-monitor.outputs.scores }}'  
```

## 🚀 Advanced Tips

### Avoid committing directly to the branch and instead generate a PR

If you have implemented the recommended branch protection rules from the OpenSSF Scorecard, committing and pushing directly to the `main` branch will be impossible. An easy alternative is to extend the pipeline to automatically generate a PR for you:

```yml
name: "OpenSSF Scoring"
on: 
  # ...

permissions:
  contents: write
  pull-requests: write
  issues: write
  packages: none

jobs:
  security-scoring:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: OpenSSF Scorecard Monitor
        uses: ossf/scorecard-monitor@v2.0.0-beta8
        id: openssf-scorecard-monitor
        with:
          auto-commit: false
          auto-push: false
          generate-issue: true
          # ....
      - name: Print the scores
        run: |
          echo '${{ steps.openssf-scorecard-monitor.outputs.scores }}'
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@38e0b6e68b4c852a5500a94740f0e535e0d7ba54 # v4.2.4
        with:
            token: ${{ secrets.GITHUB_TOKEN }}
            commit-message: OpenSSF Scorecard Report Updated
            title: OpenSSF Scorecard Report Updated
            body: OpenSSF Scorecard Report Updated
            base: main
            assignees: ${{ github.actor }}
            branch: openssf-scorecard-report-updated
            delete-branch: true
```

### Embed Report version

If you want to mix the report in markdown format with other content, then you can use `report-tags-enabled=true` then report file will use the tags to add/update the report summary without affecting what is before or after the tagged section.

This is very useful for static websites, here is [an example using docusaurus][onebeyond-report].

### Custom tags

By default we use `<!-- OPENSSF-SCORECARD-MONITOR:START -->` and `<!-- OPENSSF-SCORECARD-MONITOR:END -->`, but this can be customize by adding your custom tags as `report-start-tag` and `report-end-tag`

### Increase HTTP request in parallel

You can control the amount of parallel requests performed against the OpenSSF Scorecard Api by defining any numerical value in `max-request-in-parallel`, like `max-request-in-parallel=15`.

By default the value is 10, higher values might not be a good use of the API and you can hit some limits, please check with OpenSSF if you want to rise the limits safely.

### Exclude repos

In some scenarios we want to enable the auto-discovery mode but we want to ignore certain repos, the best way to achieve that is by editing the `scope.json` file and add any report that you want to ignore in the `excluded` section for that specific organization.

## 🍿 Other

### Scoping Structure

Just for reference, the scope will be stored this way:

File: `reporting/scope.json`

```json
{
    "github.com": {
      "included": {
        "UlisesGascon":[
          "tor-detect-middleware", 
          "check-my-headers", 
          "express-simple-pagination"
        ]
      },
      "excluded": {
        "UlisesGascon": [
          "demo-stuff"
        ]
      }
    }

}
```

### Database structure

Just for reference, the database will store the current value and previous values with the date:

```json
{
  "github.com": {
    "UlisesGascon": {
      "check-my-headers": {
        "previous": [ {
          "score": 6.7,
          "date": "2022-08-21"
        }],
        "current": {
          "score": 4.4,
          "date": "2022-11-28"
        }
      }
    }
  }
}
```

## 💪 Contributing

Please read [CONTRIBUTING.md](https://github.com/ossf/scorecard-monitor/blob/main/CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests to us. You need to accept DCO 1.1 in order to make contributions.
