
```shell
# 1. Create the repo (creates it on GitHub and adds a remote)
gh repo create influxdb-to-victoriametrics --private --description "Migration tracking: InfluxDB -> VictoriaMetrics" --clone

# this clones it locally too and cds you don't need to cd manually — 
# check with:
cd influxdb-to-victoriametrics

# 2. Add a README/initial commit if repo created empty
echo "# InfluxDB to VictoriaMetrics Migration" > README.md
git add README.md && git commit -m "Initial commit" && git push

# 3. Create a Projects v2 board and link it to the repo
gh project create --owner "@me" --title "InfluxDB → VictoriaMetrics Migration"

# gh project create returns a project number/URL — note it, you'll need
# the project number for adding items later
```

At that point you cd into the clone and launch Claude Code as usual — and since `gh` is already authenticated in that shell, Claude Code can keep using `gh issue create`, `gh project item-add`, etc. to populate the board as you work.
