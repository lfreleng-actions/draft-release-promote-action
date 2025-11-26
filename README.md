<!--
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# ⏫ Promote Draft Release

Promotes a draft GitHub release to a full release.

## draft-release-promote-action

## Usage Example

<!-- markdownlint-disable MD046 -->

```yaml
steps:
  - name: 'Promote Draft Release'
    uses: lfreleng-actions/draft-release-promote-action@main
    with:
      TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

<!-- markdownlint-enable MD046 -->

## Token Permissions

The token needs the following permissions:

`contents: write`

## Inputs

<!-- markdownlint-disable MD013 -->

| Variable Name | Required | Description                                          |
| ------------- | -------- | ---------------------------------------------------- |
| token         | True     | GitHub token with relevant permissions               |
| latest        | False    | Mark as the latest release                           |
| tag           | False    | Tag of draft release to promote                      |
| name          | False    | Name of draft release to promote                     |
| sort_by       | False    | Sort by this field (see note below for valid options |
| sort_reverse  | False    | Reverse the sort order of results                    |
| dry-run       | False    | Perform validation without promoting the release     |

<!-- markdownlint-enable MD013 -->

Valid options for sort_by input:

- createdAt
- isDraft
- isLatest
- isPrerelease
- name
- publishedAt
- tagName

## Behavior

### Dry Run Mode

When `dry-run` is `true`, the action will:

- Perform all validation logic
- Select the matching draft release
- Display warnings for non-matching draft releases
- **Skip the actual promotion** of the release
- Output what would happen

Example usage:

```yaml
- name: 'Promote draft release (dry-run)'
  uses: lfreleng-actions/draft-release-promote-action@main
  with:
    token: "${{ secrets.GITHUB_TOKEN }}"
    tag: "${{ github.ref_name }}"
    latest: true
    dry-run: true
```

Example dry-run output:

```text
Dry-run: Would promote v0.1.3 and set as latest
```

The GitHub Step Summary will show:

```markdown
## 🔍 Dry Run: v0.1.3
Would promote to latest release
```

### Draft Releases

When more than one draft release exists in the repository:

- The action will promote the draft release matching the specified `tag` (or `name`)
- The action warns about other non-matching draft releases
- The warning appears in both the console output and GitHub Actions step summary

Example warning annotation:

```text
::warning::Non-matching draft releases found: v0.1.0
```

When more than one non-matching draft exists, the action comma-separates them:

```text
::warning::Non-matching draft releases found: v0.1.0, v0.9.0
```

### Error Handling

The action provides clear error messages for common scenarios:

- GitHub token not provided (required for authentication)
- `jq` command not found (required for JSON processing)
- Invalid `sort_by` field value
- No draft releases found in the repository
- No draft release matches the specified tag or name

## Implementation Details

Uses the GitHub CLI command:

`gh release list --json createdAt,isDraft,isLatest,isPrerelease,name,publishedAt,tagName`

Example output:

```json
[
  {
    "createdAt": "2025-04-18T05:19:03Z",
    "isDraft": true,
    "isLatest": false,
    "isPrerelease": false,
    "name": "",
    "publishedAt": "0001-01-01T00:00:00Z",
    "tagName": "v9.9.3"
  },
  {
    "createdAt": "2025-04-17T11:45:14Z",
    "isDraft": false,
    "isLatest": false,
    "isPrerelease": false,
    "name": "",
    "publishedAt": "2025-04-19T05:00:04Z",
    "tagName": "v9.9.0"
  }
]
```

This is then passed through JQ commands to sort and filter the results.

Some examples:

`jq "[.[] | select(.isDraft==true)]"`

`jq "sort_by(.tagName)"`

`jq '. | reverse'`
