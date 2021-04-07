# pypi-package-stats
Github actions repo to collect pypi stats. Inspired by Simon's repo https://github.com/simonw/package-stats

Below is workflow yaml with comments on each section.

```yaml
name: Get latest package stats from pypi

# Cron to run workflow/job on schedule.
on:
  workflow_dispatch:
  schedule:
    - cron: '0 9 * * 1'

jobs:
  scheduled:
    runs-on: ubuntu-latest
    
    steps:
    # First step is to check out the repo.
    - name: Check out repo
      uses: actions/checkout@v2
      
      # Next we curl stats from api which returns json. Replace yahoo-finance-api with package of choice.
    - name: Get latest PyPI stats
      run: |
        curl "https://pypistats.org/api/packages/yahoo-finance-api/overall?mirrors=true" > stats.json
      
      # Next below uses jq to parse json and sum total number of downloads.
    - name: Sum total downloads
      run: |
        cat stats.json | jq '[.data[] .downloads] | add' > total_count.txt 
      
      # Finally workflow checks if files have changed post above steps, if true then new files are committed to the repo with timestamp, else pass.
      # I've picked this bit from Simon's repo without much deep dive.
    - name: Commit and push if s changed
      run: |-
        git config user.name "Automated"
        git config user.email "actions@users.noreply.github.com"
        git add -A
        timestamp=$(date -u)
        git commit -m "Latest data: ${timestamp}" || exit 0
        git push ```
