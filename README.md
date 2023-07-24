# Readme Translate [![Tweet](https://img.shields.io/twitter/url/http/shields.io.svg?style=social)](https://twitter.com/intent/tweet?url=https%3A%2F%2Fgithub.com%2Fcrowdin%2Freadme-translate&text=A%20GitHub%20Action%20to%20automate%20the%20translation%20of%20your%20README%20files%20via%20Crowdin)&nbsp;[![GitHub Repo stars](https://img.shields.io/github/stars/crowdin/readme-translate?style=social&cacheSeconds=1800)](https://github.com/crowdin/readme-translate/stargazers)

A GitHub Action to automate the translation of your README.md files via Crowdin ✨

[![build-test](https://github.com/crowdin/readme-translate/actions/workflows/test.yml/badge.svg)](https://github.com/crowdin/readme-translate/actions/workflows/test.yml)
[![e2e-test](https://github.com/crowdin/readme-translate/actions/workflows/e2e-test.yml/badge.svg)](https://github.com/crowdin/readme-translate/actions/workflows/e2e-test.yml)
[![codecov](https://codecov.io/github/crowdin/readme-translate/branch/main/graph/badge.svg?token=IWJHNN05QB)](https://codecov.io/github/crowdin/readme-translate)
[![GitHub](https://img.shields.io/github/license/crowdin/readme-translate?cacheSeconds=50000)](https://github.com/crowdin/readme-translate/blob/master/LICENSE)

## What does this action do?

- Upload your `README.md` file to Crowdin
- Download the translations from Crowdin to the specified place
- Manage the languages switcher in your `README.md` file

## Usage

Set up a workflow in *.github/workflows/readme-translate.yml* (or add a job to your existing workflows).

Read the [Configuring a workflow](https://help.github.com/en/articles/configuring-a-workflow) article for more details on how to create and set up custom workflows.

```yaml
# .github/workflows/readme-translate.yml
name: Readme Translate

on:
  # When you push to the `main` branch
  push:
    branches: [ main ]
  # And optionally, once every 12 hours
  schedule:
    - cron: '0 */12 * * *' # https://crontab.guru/#0_*/12_*_*_*
  # To manually run this workflow
  workflow_dispatch:

jobs:
  readme-translate:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v3

    - name: Readme Translate
      uses: crowdin/readme-translate@v0.1.0
      env:
        CROWDIN_PROJECT_ID: ${{ secrets.CROWDIN_PROJECT_ID }}
        CROWDIN_PERSONAL_TOKEN: ${{ secrets.CROWDIN_PERSONAL_TOKEN }}
        # Optional. Only for Crowdin Enterprise
        CROWDIN_ORGANIZATION: ${{ secrets.CROWDIN_ORGANIZATION }}
```

### Creating a PR

To create a PR with the updated table, we recommend usage of the [Create Pull Request](https://github.com/peter-evans/create-pull-request) Action:

```yaml
# .github/workflows/readme-translate.yml
name: Readme Translate

on:
  # When you push to the `main` branch
  push:
    branches: [ main ]
  # And optionally, once every 12 hours
  schedule:
    - cron: '0 */12 * * *' # https://crontab.guru/#0_*/12_*_*_*
  # To manually run this workflow
  workflow_dispatch:

jobs:
  readme-translate:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v3

    - name: Readme Translate
      uses: crowdin/readme-translate@v0.1.0
      env:
        CROWDIN_PROJECT_ID: ${{ secrets.CROWDIN_PROJECT_ID }}
        CROWDIN_PERSONAL_TOKEN: ${{ secrets.CROWDIN_PERSONAL_TOKEN }}

    - name: Create Pull Request
      uses: peter-evans/create-pull-request@v4
      with:
        title: New Readme Translations by Crowdin
        body: By [readme-translate](https://github.com/crowdin/readme-translate) GitHub action
        commit-message: New Readme Translations
        committer: Crowdin Bot <support+bot@crowdin.com>
        branch: docs/readme-translations
```

### Languages Switcher

The Readme Translate Action will automatically add a language switcher to your `README.md` file.

To enable it, add the following placeholders to your `README.md` file:

```
<!-- README-TRANSLATE-LANGUAGES-START -->
<!-- README-TRANSLATE-LANGUAGES-END -->
```

Also, don't forget to add the following option to your workflow step:

```diff
​- name: Readme Translate
  uses: crowdin/readme-translate@v0.1.0
  with:
+    language_switcher: true
  env:
    CROWDIN_PROJECT_ID: ${{ secrets.CROWDIN_PROJECT_ID }}
    CROWDIN_PERSONAL_TOKEN: ${{ secrets.CROWDIN_PERSONAL_TOKEN }}
```

### Options

| Option                  | Default value                 | Description                                                                                              |
|-------------------------|-------------------------------|----------------------------------------------------------------------------------------------------------|
| `file`                  | `README.md`                   | The Readme file name                                                                                     |
| `destination`           | `./`                          | A directory where the localized readmes will be placed                                                   |
| `languages`             | All Crowdin project languages | A list of Crowdin [language codes](https://developer.crowdin.com/language-codes) to translate the Readme |
| `language_switcher`     | `false`                       | Defines whether to add a language switcher to the Readme                                                 |
| `upload_sources`        | `true`                        | Defines whether to upload the source file to Crowdin                                                     |
| `download_translations` | `true`                        | Defines whether to download translations from Crowdin                                                    |

### Environment variables

| Variable                 | Description                                                                             |
|--------------------------|-----------------------------------------------------------------------------------------|
| `CROWDIN_PROJECT_ID`     | Your Crowdin Project ID (numeric)                                                       |
| `CROWDIN_PERSONAL_TOKEN` | Your Crowdin [Personal Access Token](https://support.crowdin.com/account-settings/#api) |
| `CROWDIN_ORGANIZATION`   | Your Crowdin Enterprise organization name (only for Crowdin Enterprise)                 |

### Automatic Pre-Translation

You can Pre-Translate your `README.md` file before creating a PR. Use the following configuration for that:

```yaml
# .github/workflows/readme-translate.yml
name: Readme Translate

on:
  push:
    branches: [ main ]

jobs:
  readme-translate:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Upload README.md to Crowdin
        uses: crowdin/readme-translate@v0.1.0
        with:
          upload_sources: true
          download_translations: false
        env:
          CROWDIN_PROJECT_ID: ${{ secrets.CROWDIN_PROJECT_ID }}
          CROWDIN_PERSONAL_TOKEN: ${{ secrets.CROWDIN_PERSONAL_TOKEN }}

      - name: Pre-Translate README.md via Translation Memory
        uses: crowdin/github-action@v1
        with:
          command: 'pre-translate'
          command_args: '-l uk --method tm'
        env:
          CROWDIN_PROJECT_ID: ${{ secrets.CROWDIN_PROJECT_ID }}
          CROWDIN_PERSONAL_TOKEN: ${{ secrets.CROWDIN_PERSONAL_TOKEN }}

      - name: Download Readme translations from Crowdin
        uses: crowdin/readme-translate@v0.1.0
        with:
          upload_sources: false
          download_translations: true
        env:
          CROWDIN_PROJECT_ID: ${{ secrets.CROWDIN_PROJECT_ID }}
          CROWDIN_PERSONAL_TOKEN: ${{ secrets.CROWDIN_PERSONAL_TOKEN }}

      # Create a PR with the latest translations
```

For more about the Pre-Translate command arguments, please refer to the official [Crowdin CLI documentation](https://crowdin.github.io/crowdin-cli/commands/crowdin-pre-translate).

## Contributing

If you want to contribute please read the [Contributing](/CONTRIBUTING.md) guidelines.

## Author

- [Andrii Bodnar](https://github.com/andrii-bodnar/)

## License

<pre>
The Readme Translate Action is licensed under the MIT License.
See the LICENSE file distributed with this work for additional
information regarding copyright ownership.

Except as contained in the LICENSE file, the name(s) of the above copyright
holders shall not be used in advertising or otherwise to promote the sale,
use or other dealings in this Software without prior written authorization.
</pre>
