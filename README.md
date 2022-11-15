# coverity-buildless-analysis
This will run all Coverity Analysis phases by using the Coverity buildless capture (cov-capture). This action is using [lejouni/coverity-commit-checker](https://github.com/lejouni/coverity-commit-checker) to check that is the actual Coverity commit needed or not. If there are introduced any new findings or fixed any existing ones, then commit will be done otherwise no commits.

## Prerequisities
This Github Action expects that Coverity Analysis tools are in the runner PATH. One way to get Coverity Analysis tools into the runner PATH is to run first the [lejouni/setup-coverity-analysis](https://github.com/lejouni/setup-coverity-analysis) -action.

Coverity buildless capture is supported for following coding languages and for others you have to use build capture [lejouni/coverity-build-analysis](https://github.com/lejouni/coverity-build-analysis)
| Language | Language identifier |
|---------|----------|
| Apex | apex |
| C# | cs |
| Java | java |
| JavaScript | javascript |
| TypeScript | typescript |
| PHP | php |
| Python | python |
| Ruby | ruby |

## Available Options
| Option name | Description | Default value | Required |
|----------|----------|---------|----------|
| log_level | Logging level | DEBUG | false |
| teams_webhook_url | Microsoft Teams WebHook URL. By giving this, the Teams notification is activated | - | false |
| force_commit | Setting this true, it will do the commit and will not do any checkings. | false | false |
| dryrun | Set this true, if you want to run tests and not to do the commit. | false | false |
| break_build | Set this true, if you want to break the build, if there are new findings. | false | false |
| emit_threshold | With this you can set the emit threshold percentage, the default is 95 | 95 | false |
| viewID | ID or the name of that view which result is used to get findings | - | false |
| cov_capture_mode | Which cov-capture mode is used. Options are project, scm, source (default) and config | source | false |
| cov_capture_source | Source folder where capture will be started or SCM URL | ${{github.workspace}} | false |
| cov_capture_params | Additional parameters for cov-capture phase | --no-security-da | false |
| cov_analysis_params | Additional parameters for cov-analyze phase | --webapp-security --security --strip-path=${{github.workspace}} -en HARDCODED_CREDENTIALS | false |
| cov_incremental_analysis_params | Additional parameters for cov-run-desktop phase | --set-new-defect-owner false --whole-program --ignore-uncapturable-inputs true --reference-snapshot="latest" --present-in-reference false --webapp-security --security --strip-path=${{github.workspace}} -en HARDCODED_CREDENTIALS | false |
| cov_analysis_mode | Analysis mode will tell the action that is full or incremental analysis requested, Options are full and incremental | full | false |
| github_access_token | This is required when using incremental analysis mode. This is used to get modified files via Github Api | - | In full analysis false and in incremental analysis true |

## Environment variables what this action expects

These key-value pairs must be in environment values and are accessed with **${{env.key}}**
| Key | Value | Description |
|----------|--------|---------|
| project | ${{env.project}}.| Project name in Coverity Connect |
| stream | ${{env.stream}} | Project stream name where you want to commit the findings |
| cov_url | ${{env.cov_url}} | Coverity Connect URL |
| cov_username | ${{env.cov_username}} | Username for Coverity Connect access |
| cov_password | ${{env.cov_password}} | Password for Coverity Connect access |
| cov_intermediate_dir | ${{env.cov_intermediate_dir}} | Path to intermediate directory |
| cov_output_format | ${{env.cov_output_format}} | Output format (html, sarif or json) |
| cov_output | ${{env.cov_output}} | Output file with full path and if output format is html then this must the folder where html will be created |

## Usage examples
Run the Coverity full buildless analysis with source mode.
```yaml
    - if: ${{github.event_name == 'pull_request'}}
      name: Build with Maven and Full Analyze with Coverity # This will run the full Coverity Analsysis
      uses: lejouni/coverity-buildless-analysis@v2.8.29
```
Run the Coverity full buildless analysis with project mode.
```yaml
    - if: ${{github.event_name == 'pull_request'}}
      name: Build with Maven and Full Analyze with Coverity # This will run the full Coverity Analsysis
      uses: lejouni/coverity-buildless-analysis@v2.8.29
      with:
        cov_capture_mode: project # Options are project, scm, source (default) and config
```
Run the Coverity incremental buildless analysis with project mode.
```yaml
    - if: ${{github.event_name == 'push'}}
      name: Build with Maven and Incremental Analyze with Coverity # This will run the incremental Coverity Analsysis
      uses: lejouni/coverity-buildless-analysis@v2.8.29
      with:
        cov_capture_mode: project # Options are project, scm, source (default) and config
        cov_analysis_mode: incremental # Optional, but options are full (default) or incremental
        github_access_token: ${{secrets.ACCESS_TOKEN_GITHUB}} # this is required in incremental mode, used to get changed files via Github API
```
Full pipeline example by using the [lejouni/setup-coverity-analysis](https://github.com/lejouni/setup-coverity-analysis) -action to set up the Coverity Analysis tools first.
```yaml
name: Java CI with Maven and Coverity

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3 # This will checkout the source codes from repository

    - name: Set up JDK 1.11 # This will add Java into the runners PATH
      uses: actions/setup-java@v3.6.0
      with:
        java-version: '11'
        distribution: 'temurin'
        cache: 'maven'

    - name: Set up Coverity # This will add Coverity Analysis tools into runner PATH
      uses: lejouni/setup-coverity-analysis@v2.8.20
      with:
        cov_version: cov-analysis-linux64-2022.6.1
        cov_url: ${{secrets.COVERITY_SERVER_URL}} # Coverity Connect server URL
        cov_license: ${{github.workspace}}/scripts/license.dat
        cov_username: ${{secrets.COVERITY_USERNAME}} # Coverity Connect username
        cov_password: ${{secrets.COVERITY_ACCESS_TOKEN}} # Coverity Connect password
        cov_output_format: sarif # Optional, but if given the options are html, json and sarif
        cov_output: ${{github.workspace}}/coverity_results.sarif.json
        project: test-project # Project name can be given, but if not, then repository name is used as a project name
        stream: test-project-main # Stream can be give as well, but if not, then repository name-branch name is used.
        create_if_not_exists: true # will create project and stream if they don't exists yet
        cache: coverity # Optional, but if given the options are coverity, idir and all

    - if: ${{github.event_name == 'pull_request'}}
      name: Build with Maven and Full Analyze with Coverity # This will run the full Coverity Analsysis
      uses: lejouni/coverity-buildless-analysis@v2.8.29
      with:
        cov_capture_mode: project # Options are project, scm, source (default) and config

    - if: ${{github.event_name == 'push'}}
      name: Build with Maven and Incremental Analyze with Coverity # This will run the incremental Coverity Analsysis
      uses: lejouni/coverity-buildless-analysis@v2.8.29
      with:
        cov_capture_mode: project # Options are project, scm, source (default) and config
        cov_analysis_mode: incremental # Optional, but options are full (default) or incremental
        github_access_token: ${{secrets.ACCESS_TOKEN_GITHUB}} # this is required in incremental mode, used to get changed files via Github API

    - name: Upload SARIF file
      uses: github/codeql-action/upload-sarif@v2
      with:
        # Path to SARIF file
        sarif_file: ${{github.workspace}}/coverity_results.sarif.json
      continue-on-error: true

    - name: Archive scanning results
      uses: actions/upload-artifact@v3
      with:
        name: coverity-scan-results
        path: ${{github.workspace}}/coverity_results.sarif.json
      continue-on-error: true
```
