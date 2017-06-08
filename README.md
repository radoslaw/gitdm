Please use `*.sh` scripts to run analytics (`all*.sh` for full analysis and `rels*.sh` for per release stats)
This program assumes that gitdm lives in: `~/dev/cncf/gitdm/` and kubernetes in `~/dev/go/src/k8s.io/kubernetes/`
Output files are placed in `kubernetes/` directory.

This is an iterational process:
Run any of scripts. Review its output in kubernetes directory. Iteratively adjust mappings to handle more authors (config/mappings is in
`cncf-config/`)

You can also run via `./debug.sh` to halt in debugger and review hackers structure and those who were not found. See `cncfdm.py`:`DebugUnknowns`

Final report:
Data: https://docs.google.com/spreadsheets/d/15otmXVx8Gd6JzfiGP_OSjP8M9zyLeLof5-IGQKEb0UQ/edit?usp=sharing
Report: https://docs.google.com/document/d/1RKtRamlu4D_OpTDFTKNpMsmV51obdZlPWbXVj-LrDuw/edit?usp=sharing

# Contributing

Pull Request are welcome.
Our mapping is not complete, please see config files in: https://github.com/cncf/gitdm/blob/master/cncf-config/.

File https://github.com/cncf/gitdm/blob/master/cncf-config/email-map is a direct mapping email to employer.

There is also a long list of unknown emails, please see section `Developers with unknown affiliation`:
https://github.com/cncf/gitdm/blob/master/results.txt

All unknown developers have 4 or less contribution.
Current list of unknown developers (names & emails): https://github.com/cncf/gitdm/blob/master/unknown_devs.txt

# Detailed description:

Use `./rerun_data.sh` to regenerate all data which means:
- Data for `kubernetes/kubernetes` repository (all time) with 3 mappings of Unknown developers: no mapping (list them with their email & name), map them to theirs email domain's (`user@gmail.com` --> `'Gmail *'`), map all of them to '(Unknown)'. This is done via running: (`./all.sh`, `./all_no_map.sh`, `./all_with_map.sh`). Output comes to `./kubernetes/all_time/` directory
- Data for `kubernetes/kubernetes` repository divided into releases v1.0.0, v1.1.0, ..., v1.6.0 (with 3 types of mappings described above). This is done with (`./rels.sh`, `./rels_strict.sh`, `./rels_no_map.sh`). Output comes to ./kubernetes/v1.X.0-v1.Y.0/` directory: X=0,1,2,3,4,5, Y=1,2,3,4,5,6)
- After those 2 steps we need to analyse `cncfdm.py` outputs, it is done via calling: `./analysis_all.sh` (alanyses all time results) and then `./analysis_rels.sh` (for per release data)
- Data for all 90 (currently) repos that makes entire Kubernetes project with `./kubernetes_repos.sh` script. It will be descibed later.

Final files generated by first 2 calls (for single repo kubernetes/kubernetes) are in `./kubernetes/all_time/*.txt` and `./kubernetes/v1.X.0-v1.Y.0/*.txt`
All scripts are configured to ignore commits related to files from `contrib` directory. This is because external sources are placed here, and many commits are just adding external libraries which would make data less acurate

All of them use `git log` call with specific args piped to `cncfdm.py` call with specific parameters. See `./run.sh` for example, all other calls use the same commands `git log` and `cncfdm.py` with other parameters.
To see parameters for `cncfdm.py` see comments inside `cncfdm.py` file describing all possible options. For more details about how `cncfdm.py` tool works refer to its sources and other `*.py` files.

Those files are analysed by `./analysis_all.sh` and `./analysis_rels.sh`. First just calls:
`ruby analysis.rb all kubernetes/all_time/first_run_patch.txt kubernetes/all_time/run_no_map_patch.txt kubernetes/all_time/run_with_map_patch.txt`
Second calls:
`ruby analysis.rb v1.0_v1.1 kubernetes/*/output_strict_patch.txt kubernetes/*/output_patch.txt kubernetes/*/output_no_map_patch.txt`
This ruby tool expects to get 3 files (one with no unknown developers mapping, 2nd with mapping to domain name and 3rd with mapping to (Unknown).

Output of this analysis.rb tool is in `project/<prefix>_<key>_<type>`.csv files.
<prefix>: can be `all` or `v1.X.0-v1.Y.0` - it means that file is for all time data or for specific release of `kubernetes/kubernetes`
<key>: can be changeset, employers, lines, signoffs - it means file contains data sorted by this <key> desc.
<type>: can be `sum`, `top`, `all`:
- `all` means that file conatins all data for given <prefix> sorted by <key> desc (header is: `idx,company,n,percent` which means n-th, company name, n developers, % all developers) `All known` is sum of all detected developers
- `top` means that there will be top 10 data from `all` but also must contain data for: '(Unknown)', 'Gmail *', 'Qq *', 'Outlook *', 'Yahoo *', 'Hotmail *', '(Self)', '(Not Found)'. Header is the same as in `all`.
- `sum` contains summary value for all found developers. It has other header `N companies,sum,percent`: numer of developer's companies found, sum of <key> for all found developers, % of sum <key> as a part of sum <key> for all developers.
- Special names: `All known` (sum all known developers), `(Self)` (developers working on their own), `(Not Found)` (developers for whom I was searching in multiple sources and wasn't able to determine their employer), `(Unknown)` (developers not mapped (yet?)), `Somename *` (sum of developers having emails on `Somename` domain). Asterisk `*` added to indicate this.

This data is directly used for "Who writes Kubernetes" report.

`./kubernetes_repos.sh` script is used to generate all time data for all kubernetes repos. To use it You must have all kubernetes repositories (90 from 6 different orgs) cloned in` ~/dev/go/src/k8s/`.
Orgs are: kubernetes, kubernetes-incubator, kubernetes-client, kubernetes-contrib, kubernetes-cluster-automation and kubernetes-ui.
It generates statistics for each single repo via:
`./anyrepo.sh ~/dev/go/src/k8s.io/<repo-name> <repo-name>`
See details in `./kubernetes_repos.sh`.
<repo-name> is a directory where given kubernetes repository is cloned.
To clone repository do:
`cd ~/dev/go/src/k8s/`
`https://github.com/<one-of-6-kubernetes-orgs>/<kubernetes-repo-name>.git`
<one-of-6-kubernetes-orgs>: kubernetes, kubernetes-incubator, kubernetes-client, kubernetes-contrib, kubernetes-cluster-automation and kubernetes-ui
<kubernetes-repo-name: please lookup all repo names in all kubernetes orgs at GitHub.
`./anyrepo.sh` just calls `cncfdm.py` with appropriate args (like exclude vendor dir numstat etc).
There is also `./anyreporange.sh` that allows to query repo for given time range (`cncfdm.py` supports that too).
Output of this is in `repos/<repo-name>.<ext>`
<repo-name>: repository name `./anyrepo.sh` was called with.
<ext>: txt, csv, html, out: txt: main data file, csv: dump list of employers in given repo, html: the same as txt but in HTML format, out: `cncfdm.py` verbose output messages (for debugging)

Finally `./kubernetes_repos.sh` calls:
`./multirepo.sh` with all 90 repositories directories listed.
It gathers `git log` on each of them and concatenates all those files and then run `cncfdm.py` on concatenated result (see `./multirepo.sh`)
Results are saved to `repos/combined.<ext>` <ext> is the same as for `anyrepo.sh`.

Typicsal work looks like reruning `./kubernetes_repos.sh` and examine `repos/combined.txt` for unknown developers.
Research on google, clearbit, github, LinkedIn, Facebook whatever -> update `cncf-config/<filename>` and re-run `./kubernetes_repos.sh`
<filename>: usually in this order: email-map, domain-map, a in very rare cases: aliases, gitdm.config-cncf or group mappings in groups/

Also when running data for single `kubernetes/kubernetes` for example with `./all.sh` examining developers found in `./kubernetes/all_time/first_run_patch.txt`.

After all this data is generated, `./kubernetes_repos.sh` concatenates all single repo data into single output file: `repos/merged.out` just to allow browsing all data in single file.
It also generates developers and companies statistics via:
`./topdevs.sh` call.
It calls ruby tool on combined output of all 90 kubernetes repos (saved as CSV):
`ruby topdevs.rb repos/combined.csv`
That tool generates files:
`companies_by_name.csv` - this is a list of companies found sorted byb their names (no case sensitive) to allow manual examination if we don't have duplicates with slightly different names like "Google" vs "Googe Corporation" vs "Google Corp." or "google"
`companies_by_count.csv` - list of coompanies found sorted (desc) by number of employers. Serves similar purpose but from different perspective.
`unknown_devs.txt`, `unknown_devs.csv`, `unknown_emails.csv` - list of developers for who we don't have a mapping. Used to priritize searching for devs, and `unknown_emails.csv` is in format requested by clearbit batch.
There are clearbit tools in `clearbit_tools/` directory. Look for any files with `.rb` extension. I've already did 3 rounds of commercial requests to clearbit. And they returned quite a lot of data. But those files are not checked in and are listed in `./.gitignore` because we have to pay for that data.
Those tools are used to enrich of `cncf-config/email-map` mapping.
`google_other.txt` - contains list of Google developers with email on domain different than `@google.com`.
`./changesets.csv`, `./added.csv`, `./removed.csv` files contains developers sorted by changesets, added lines, removed lines desc. This is used to generate Top N developers in given criteria.

`./new_devs.sh` (also used by `./rerun_data.sh`) is used to generate statistics about new developers between `kubernetes/kubernetes` releases.
It calls: `ruby new_devs.rb kubernetes/v1.X.0-v1.Y.0/output_strict_patch.csv` for all X and Y.
`new_devs.rb` just generates information about developers who were new between each releases and file `new_devs.csv` which contains list of companies who introduced most new developers overall (sorted by # of new developers desc).

That covers typical usage and data for "Who writes Kubernetes report"

Other tools include:
`see_parser.sh` - display data feed as used by `cncfdm.py` tool
`range.sh` - generate stats for `Linux kernel` for given data range (1st and 2nnd commandline argument like 2016-01-01 2017-01-01), assumes Linux repo (`torvalds/linux`) is cloned in `~/dev/linux/`
`range_<period>.sh` - used to generate monthly, quarterly, yearly stats using above `./range.sh`, for example `./range_monthly.sh`.

To work on Prometheus contributors before and after joining CNCF:
Prometheus joined CNCF at 2016-05-09.
You need to clone all prometheus repos into `~/dev/prometheus` using `./clone_prometheus.sh`
Then You need to get number of distinct Prometheus contributors before joining CNCF:
./prometheus_repos.sh 2015-05-09 2016-05-08 ~/dev/prometheus/
Result is:
```
Processed 2721 csets from 230 developers
252 employers found
A total of 1558445 lines added, 353900 removed (delta 1204545)
```
Now let's check numbe rof distinct contributors after 2016-05-09:
`./prometheus_repos.sh 2016-05-09 2017-06-01 ~/dev/prometheus/`
```
Processed 2817 csets from 346 developers
365 employers found
A total of 2696196 lines added, 771502 removed (delta 1924694)
```

We have an increase from 230 to 365 which is 59% more.

# Report
Links to data and generated report are here: `./res/links.txt`

# CNCF Projects join statistics
- CNCF Projects joindates are: https://github.com/cncf/toc#projects
- To generate statistics for Prometheus 90 days before joining CNCF and 90 days after joining try this:
- Run: `./clone_prometheus.sh`
- Run `./cncf_join_analysis.sh prometheus 2016-05-09 90 ~/dev/prometheus/`
- Result in `prometheus_repos/result.txt`

- Create directory where You want to put links to kubernetes repos, like this: `mkdir ~/dev/kubernetes_repos_links`
- Now copy kubernetes_repos.sh to link_kubernetes_repos.sh: `cp kubernetes_repos.sh link_kubernetes_repos.sh`
- Open copy and add 1st line: `cd ~/dev/kubernetes_repos_links`
- Now replace lines like `./anyrepo.sh ~/dev/go/src/k8s.io/test-infra/ test-infra` with ln -s ~/dev/go/src/k8s.io/test-infra/ test-infra`, run it, voila, You have k8s repos links in `~/dev/kubernetes_repos_links`
- Now command that takes on Kubernetes repos should be: `./cncf_join_analysis.sh kubernetes 2016-03-10 90 ~/dev/kubernetes_repos_links`
- Result in `kubernetes_repos/result.txt`

- To generate statistics for OpenTracing 90 days before joining CNCF and 90 days after joining try this:
- Run: `./clone_opentracing.sh`
- Run `./cncf_join_analysis.sh opentracing 2016-08-17 90 ~/dev/opentracing/`
- Result in `opentracing_repos/result.txt`

- There is also All-in-one script to regenerate all CNCF Projects join statistics: run: `./join_stats.sh`

# Typical update of "Who writes Kubernetes report"
- Run ./pull_kubernetes.sh to get all Kubernetes repos updated.
- Go to `cd dev/go/src/k8s.io/kubernetes/` and update this repository as well.
- New release since last run (1.7) so many scripts needs to be updated, also all repos from 3 kubernetes orgs are now in  ~/dev/kubernetes/repos so `./kubernetes_repos.sh` script need update too
- Updated `kubernetes_repos.sh` script to get repos from `~/dev/kubernetes_repos/`
- Script to regenerate all data is `./rerun_data.sh`, it needs to be investigated to support v1.7.0
- Now report is: https://docs.google.com/document/d/1RKtRamlu4D_OpTDFTKNpMsmV51obdZlPWbXVj-LrDuw/edit?usp=sharing
- Repord data sheet/draft is: https://docs.google.com/spreadsheets/d/15otmXVx8Gd6JzfiGP_OSjP8M9zyLeLof5-IGQKEb0UQ/edit#gid=0
- Now report sections:
```
Since kubernetes project started in June 2014 - 2623 Developers from 789 Companies were working on it (counting Kubernetes and all its projects 68 repos from 3 orgs).
A total of 28.4 million lines were added, 16.3 million removed.
```
Taken from: `./repos/combined.txt`
```
Processed 59041 csets from 2623 developers
789 employers found
A total of 28440262 lines added, 16342872 removed (delta 12097390)
```
For single kubernetes/kubernetes repo we have data: `kubernetes/all_time/first_run_numstat.txt`
```
Processed 28225 csets from 1338 developers
400 employers found
A total of 6667288 lines added, 4132224 removed (delta 2535064)
```
- Now how to fill data sheet/chart:
1. Sheet "all time data":
- `analysis_all_repos.sh`, generates files starting with: `report/all_repos_rest`
- `report/prefix_key_type` (prefix: all - for kubernetes/kubernetes, all_repos - for all repos, v1.x for releases), project/<prefix>_<key>_<type>
- Commits info is in `other_repos/all_kubernetes_dtfrom_dtto` and `other_repos/kubernetes_dtfrom_dtto` (for all k8s repos and kubernetes/kubernetes alone)
- To see commits for all kubernetes repos combined for last year & for last 12 months (each) separately: `grep -HIn "csets from" other_repos/all_kubernetes_range_unknown_201*`
- The same for `kubernetes/kubernetes` repo: `grep -HIn "csets from" other_repos/kubernetes_range_unknown_201*`
- Update report and report data sheet with those results
- Number of github events etc - from `cncf/velocity:projects/unlimited.csv` (this is for 201606-201705)
- And number for May 2017 is: `cncf/velocity:projects/cncf_projects_201705.csv`
```
activity,comments,prs,commits,issues,authors
Last year: 308313,217684,46351,16000,28278,1728
Last month: 30227,21371,4645,1741,2470,451
```
- Analysis for kubernetes/kubernetes (main repo) are in format: `report/all_{key}_top.csv`, import them to 2nd sheet
- Big summaries like all developers etc in `./repos/combined.txt`, for main k8s repo: `kubernetes/all_time/first_run_numstat.txt`
- Top developer stats are here: `stats/all_key.csv` (for all repos), `stats/kubernetes_key.csv` (for main repo) and `stats/v1.x_key.csv` per versions.
- Import those to last 3 sheets in data sheet
- Per verion data: `report/v1.x_v1.y_key_top.csv`, key: changesets, lines, developers, import to data sheet for all versions: 7 x 3 = 21 imports
