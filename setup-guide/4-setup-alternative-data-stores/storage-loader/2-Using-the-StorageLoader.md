[**HOME**](Home) > [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) > [Step 4: setting up alternative data stores](Setting-up-alternative-data-stores) > [1: Installing the StorageLoader](1-Installing-the-StorageLoader) > 2: Using the StorageLoader

1. [Overview](#usage-overview)
2. [Command-line options](#cli-options)
3. [Running](#running)
4. [Troubleshooting](#troubleshooting)
5. [Next steps](#next-steps)

<a name="usage-overview"/>

## 1. Overview

Running the StorageLoader is very straightforward - please review the
command-line options in the next section.

<a name="cli-options"/>

## 2. Command-line options

The StorageLoader is an executable jar:

    $ ./snowplow-storage-loader

The command-line options for StorageLoader look like this:

    Specific options:
        -c, --config CONFIG              configuration fgle
        -b CONFIG,                       base64-encoded configuration string
            --base64-config-string
        -t, --targets TARGETS_DIR        targets directory
        -i, --include compupdate,vacuum  include optional work step(s)
        -s download|delete,load,shred,analyze,archive_enriched,
            --skip                       skip work step(s)
        -r, --resolver RESOLVER          Iglu resolver config file

    Common options:
        -h, --help                       Show this message
        -v, --version                    Show version

A note on the `--include` option: this includes optional work steps
which are otherwise not used. `--include vacuum` runs a `VACUUM`
operation on the table following the load. `--include compupdate` runs
the load then determines the best compression encoding format to use for
each each of the fields in your Redshift event table, using the `:comprows:`
setting for the sample size. For more information on Amazon’s comprows
functionality, see the [Redshift documentation][comprows].

A note on the `--skip` option: this skips the work steps listed. So, for example `--skip download,load` would only run the final archive step. This is useful if you have an error in your load and need to re-run only part of it.

Instead of using the --config option, you can pass the configuration to the EmrEtlRunner via stdin. You need to set `--config -` to signal that the config is to be read from stdin rather than from a file:

```sh
$ cat config/config.yml | ./snowplow-storage-loader --config -
```

<a name="running"/>

## 3. Running

As per the above, running StorageLoader is a matter of populating
your storage [targets configurations][storage-targets] (`config/targets/`) and [configuration file][common-configuration], (`config/config.yml`) for this
example, and then invoking StorageLoader like so:

    $ ./snowplow-storage-loader --config config/config.yml --resolver config/resolver.json --targets config/targets/ --skip analyze

The `--skip analyze` is required because in Redshift only the table owner or superuser can ANALYZE a table, not our `storageloader` user.

<a name="troubleshooting" />

## 4. Troubleshooting

### locate command missing

StorageLoader depends on Snowplow's [Infobright Ruby Loader][irl], which in turn uses the `locate` shell command. If your shell complains that this is missing, in which case you can install it separately.

To install and configure `locate` on Debian/Ubuntu:

    $ sudo apt-get install mlocate
    $ sudo updatedb

<a name="next-steps" />

## Next steps

All done? Then [schedule the StorageLoader](3-Scheduling-the-StorageLoader) to regularly migrate new data into your data store (e.g. Infobright or Redshift).

[storage-targets]: https://github.com/snowplow/snowplow/wiki/Configuring-storage-targets
[common-configuration]: https://github.com/snowplow/snowplow/wiki/Common-configuration
[comprows]: http://docs.aws.amazon.com/redshift/latest/dg/r_COPY.html
