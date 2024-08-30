# npm Documentation

[![Publish](https://github.com/npm/documentation/actions/workflows/publish.yml/badge.svg)](https://github.com/npm/documentation/actions/workflows/publish.yml)

This is the documentation for [https://docs.npmjs.com/](https://docs.npmjs.com/).

[This repository](https://github.com/npm/documentation) contains the content for our documentation site, and the GitHub Actions workflows that generate the site itself.

## Quick start

1. `npm install` to download gatsby, our theme, and the dependencies
2. `npm run develop`: starts the test server at `http://localhost:8000`.
3. Update the content - it's Mdx, which is like markdown - in the `content` directory.
4. Review your content at `http://localhost:8000`. (Gatsby watches the filesystem and will reload your content changes immediately.)
5. Once you're happy, commit it and open a pull request at https://github.com/npm/documentation.
6. A CI workflow run will publish your PR to a GitHub Preview Page.
7. Once the content is reviewed, merge the pull request. That will [deploy the site](https://github.com/npm/documentation/actions/workflows/publish.yml).

Do you want to know more? Check out our [contributing guide](CONTRIBUTING.md).

## License

The npm product documentation in the content, and static folders are licensed under a [CC-BY 4.0 license](LICENSE).

All other code in this repository is licensed under a [MIT license](LICENSE-CODE).

When using the GitHub logos, be sure to follow the [GitHub logo guidelines](https://github.com/logos).
[README (1).md](https://github.com/user-attachments/files/16813690/README.1.md)
[postgres.md](https://github.com/user-attachments/files/16813687/postgres.md)

Navigation Menu
npm
/
documentation

Code
Issues
38
Pull requests
9
Actions
Projects
Security
Insights
CI
chore: update versions in publish action #1166
Jobs
Run details
Workflow file for this run
.github/workflows/ci.yml at 5df9477
# This file is automatically added by @npmcli/template-oss. Do not edit.

name: CI

on:
  workflow_dispatch:
  pull_request:
    paths-ignore:
      - cli/**
  push:
    branches:
      - main
    paths-ignore:
      - cli/**
  schedule:
    # "At 09:00 UTC (02:00 PT) on Monday" https://crontab.guru/#0_9_*_*_1
    - cron: "0 9 * * 1"

jobs:
  lint:
    name: Lint
    if: github.repository_owner == 'npm'
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: bash
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Git User
        run: |
          git config --global user.email "npm-cli+bot@github.com"
          git config --global user.name "npm CLI robot"
      - name: Setup Node
        uses: actions/setup-node@v4
        id: node
        with:
          node-version: 22.x
          check-latest: contains('22.x', '.x')
          cache: npm
      - name: Install Latest npm
        uses: ./.github/actions/install-latest-npm
        with:
          node: ${{ steps.node.outputs.node-version }}
      - name: Install Dependencies
        run: npm i --no-audit --no-fund
      - name: Lint
        run: npm run lint --ignore-scripts
      - name: Post Lint
        run: npm run postlint --ignore-scripts
      - name: Check Format
        run: npm run format:check --ignore-scripts --if-present

  test:
    name: Test - ${{ matrix.platform.name }} - ${{ matrix.node-version }}
    if: github.repository_owner == 'npm'
    strategy:
      fail-fast: false
      matrix:
        platform:
          - name: Linux
            os: ubuntu-latest
            shell: bash
        node-version:
          - 22.x
    runs-on: ${{ matrix.platform.os }}
    defaults:
      run:
        shell: ${{ matrix.platform.shell }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Git User
        run: |
          git config --global user.email "npm-cli+bot@github.com"
          git config --global user.name "npm CLI robot"
      - name: Setup Node
        uses: actions/setup-node@v4
        id: node
        with:
          node-version: ${{ matrix.node-version }}
          check-latest: contains(matrix.node-version, '.x')
          cache: npm
      - name: Install Latest npm
        uses: ./.github/actions/install-latest-npm
        with:
          node: ${{ steps.node.outputs.node-version }}
      - name: Install Dependencies
        run: npm i --no-audit --no-fund
      - name: Add Problem Matcher
        run: echo "::add-matcher::.github/matchers/tap.json"
      - name: Test
        run: npm test --ignore-scripts

  test-cli-content:
    name: Test CLI Content - ${{ matrix.platform.name }} - ${{ matrix.node-version }}
    if: github.repository_owner == 'npm'
    strategy:
      fail-fast: false
      matrix:
        platform:
          - name: Linux
            os: ubuntu-latest
            shell: bash
        node-version:
          - 22.x
    runs-on: ${{ matrix.platform.os }}
    defaults:
      run:
        shell: ${{ matrix.platform.shell }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Git User
        run: |
          git config --global user.email "npm-cli+bot@github.com"
          git config --global user.name "npm CLI robot"
      - name: Setup Node
        uses: actions/setup-node@v4
        id: node
        with:
          node-version: ${{ matrix.node-version }}
          check-latest: contains(matrix.node-version, '.x')
          cache: npm
      - name: Install Latest npm
        uses: ./.github/actions/install-latest-npm
        with:
          node: ${{ steps.node.outputs.node-version }}
      - name: Install Dependencies
        run: npm i --no-audit --no-fund
      - name: Check CLI Documentation
        run: npm run build -w cli -- --check-only
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  licenses:
    name: REUSE Compliance Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: REUSE Compliance Check
        uses: fsfe/reuse-action@v1
