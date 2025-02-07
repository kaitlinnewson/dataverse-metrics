# dataverse-metrics

## Introduction

dataverse-metrics includes two applications that report metrics

- a global metrics app that aggregates metrics from multiple Dataverse installations (v4.9+) and visualizes them on a web page.
- an installation-level metrics app (v5.1+) that can show metrics for a Dataverse instance or for any sub-Dataverse within that instance

The global dataverse-metrics app was designed to aggregate metrics across all installations of Dataverse around the world, but can also be configured for a single installation of Dataverse. It provides information on the scope and scale of holdings and activities in terms of number of file downloads.  It makes use of the Dataverse [Metrics API][] that was added in Dataverse 4.9. This app also provides graphs showing the distribution of Dataverse versions being run across installations (from v4.6 on). It relies on locally cached information (generated by python scripts) that must be periodically refreshed (e..g via a cron job).

The installation-level app leverages enhancements to the Dataverse Metrics API introduced in v5.1 to provide more detailed information about a single installation of Dataverse. It adds additional graphs showing the distribution of files by content type, 'Make Data Counts' metrics, and the number of 'unique downloaders' per dataset. All of these metrics can be displayed for the entire repository or for any sub-Dataverse within the repository, with the selection possible via a tree widget showing the overall structure of the repository, or programatically by adding a query parameter for the specific sub-Dataverse to the URL for the app. All outputs are also available in comma-separated-value (CSV) format via download buttons associated with each graph. This app does not use any local cache and doesn't require python or a cron job to run.

Both apps report metrics only for published content and hence they do not require any login or Dataverse credentials to access.

## Requirements

- Apache web server or similar
- a web browser
- Global app only: Python 2 or Python 3

## Installation

### Put code into place

Change to the parent directory for where you will install dataverse-metrics. `/var/www/html` is the default "DocumentRoot" for Apache on CentOS (defined in `/etc/httpd/conf/httpd.conf`) and is suggested as a place to install dataverse-metrics, but you are welcome to install it wherever you want and use any server you want.

    cd /var/www/html

Clone the repo:

    git clone https://github.com/IQSS/dataverse-metrics.git

Change to the directory you just created by cloning the repo:

    cd dataverse-metrics

### Configuration - Installation-level app

Copy `config.local.json.sample` to `config.local.json` and edit the following values:

- `installationURL` - the URL for the Dataverse instance, e.g. "https://demo.dataverse.edu", can be "" if the app is deployed on the same server as Dataverse itself
- `installationName` - the name of the Dataverse Repository, e.g. "Harvard Dataverse"

and optionally:

-  `dataverseTerm` - defaults to "Dataverse" - used in the app to refer to sub-dataverses, e.g. using "Collection" would result in the app showing "Click a sub-Collection name to see its metrics".
-  `maxBars` - default is 100 - the maximum number of datasets to show in the uniquedownloads by PID display. That graph is ordered by number of counts, so setting the maxBars limits the graph to the 'top <N>' results (for visibility, the CSV download will contain the full results)
- graph colors can also be changed in the config file.

### Configuration - Global app
Copy `global/config.json.sample` to `global/config.json` and edit the following values:

- `installations`: An array of Dataverse installation URLs.
- `api_response_cache_dir`: Fully qualified directory where JSON files representing API responses will be stored.
- `aggregate_output_dir`: Fully qualified directory where TSV output files of aggregated metrics will be stored.
- `num_months_to_process`: For monthly metrics, the number of months to go back in time to download metrics from each Dataverse installation.
- `month_filter_enabled`: If you have `num_months_to_process` set high (e.g. 36 months) the dates will disappear from the bottom of the bar charts. Changing this boolean to true will make only even months appear and with fewer bars the dates will be visible.
- `endpoints`: An array of Metrics API endpoints to process. Note that the two types are `single` (i.e. `datasets/bySubject`) and `monthly` (i.e. `downloads/toMonth`). (You will notice a third type called `monthly_itemized` in `config.json.sample` but it is not yet supported.)
- `blacklists`: Arrays of terms to blacklist. Only the `datasets/bySubject` endpoint can have a blacklist.
- `colors`: A single color for bar charts and a palette of colors for tree maps.
- `github_repos`: An array of GitHub repos such as `https://github.com/IQSS/dataverse`. A line will be added per repo about the number of contributors.

##### Aggregating global metrics

Now that your `global/config.json` file is ready, run the `global/metrics.py` script to create a TSV file for each of the `endpoints` and a `global/contributors.json` file for the `github_repos`, all of which will be placed in the `aggregate_output_dir` directory:

    python3 metrics.py

(Please note that if you don't have Python 3 installed, Python 2 should work fine too but Python 3 is highly recommended because Python 2 will not be maintained past January 1, 2020 according to https://pythonclock.org and [PEP 373][].)


##### Adding additional installations

The list of Dataverse installations depends on `global/all-dataverse-installations.json` which can be updated with the following script as new installations are added to the [map][] produced by [dataverse-installations][]:

    ./global/update-all-installations-list.sh

##### Updating Global Metrics

To update your metrics periodically, you'll want to queue up a shell script in some flavor of [cron][].

Here's an [example shell script](update_metrics.sh) to get you started.

On a Red Hat or CentOS system, you might drop a file like [global/update_metrics.cron](global/update_metrics.cron) into /etc/cron.d/ to update on a specified schedule.

### Viewing the visualizations

Using the instructions above, index.html files have been placed at 

- for the installation-level app: /var/www/html/dataverse-metrics/index.html
- for the global app: /var/www/html/dataverse-metrics/global/index.html 

and should be available on your Apache server at http://example.com/dataverse-metrics/index.html and http://example.com/dataverse-metrics/global/index.html respectively

## Contributing

We love contributors! Please see our [Contributing Guide](CONTRIBUTING.md) for ways you can help and check out the to do list below.

## To Do

- Drop support for Python 2. See https://python3statement.org

##Links

[![Build Status](https://travis-ci.org/IQSS/dataverse-metrics.svg?branch=master)](https://travis-ci.org/IQSS/dataverse-metrics)

[Metrics API](http://guides.dataverse.org/en/latest/api/metrics.html)

[map](https://dataverse.org/installations)

[dataverse-installations](https://github.com/IQSS/dataverse-installations)

[cron](https://en.wikipedia.org/wiki/Cron)

[Contributing Guide](CONTRIBUTING.md)

[PEP 373](https://www.python.org/dev/peps/pep-0373/)
