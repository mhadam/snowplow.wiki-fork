[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » Batch Pipeline Steps

## Dataflow diagram

[[/images/batch_pipeline_steps.png]]

## Recovery steps

The below table summarizes the actions to be taken at each particular step failure from the dataflow diagram above.

Failed step | Recovery actions 
:---:|---
 1 | If no files have been moved yet (`raw:processing` [A] is empty), rerun the *EmrEtlRunner* as usual. If (on the other hand) some files have already been moved, rerun the *EmrEtlRunner* with `--skip staging` option to proceed with processing of those log files.
 2 | Rerun the *EmrEtlRunner* with `--skip staging` option.|
 3 | Rerun the *EmrEtlRunner* with `--skip staging` option.<br><br>**Note**: The `enriched:bad` [D] and `enriched:error` [E] could contain the files produced as a result of the step 3. Therefore rerunning the *EmrEtlRunner* could result in duplicated `bad`/`error` files. This could be significant if `elasticsearch` step [8-9] is engaged for examining `bad` data [D]. The outcome would be the same data timestamped with different time values by different EMR runs.
 4 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with `--skip staging` option.
 5 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with `--skip staging` option.
 6 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with `--skip staging` option.<br><br>**Note**: The `enriched:bad` [D] and `shredded:bad` [H] could contain the files produced as a result of the step 3 and 6 respectively. Therefore rerunning the *EmrEtlRunner* could result in duplicated `bad` files. This could be significant if `elasticsearch` step (8-9) is engaged for examining `bad` data ([D],[H]). The outcome would be the same data timestamped with different time values by different EMR runs.
 7 | Delete `enriched:good` [F] and `shredded:good` [K]. Rerun the *EmrEtlRunner* with `--skip staging` option.
 8 | If duplicated `bad` data is not critical rerun the *EmrEtlRunner* with `--skip staging,emr` option. If duplicated bad data **is** critical, *instructions to come ([#2593](https://github.com/snowplow/snowplow/issues/2593))*.
 9 | If duplicated `bad` data is not critical rerun the *EmrEtlRunner* with `--skip staging,emr` option. If duplicated bad data **is** critical, *instructions to come ([#2593](https://github.com/snowplow/snowplow/issues/2593))*.
 10 | Rerun the *EmrEtlRunner* with `--skip staging,emr,elasticsearch` option.
 11 | The data load cannot result in partial load due to the use of `COMMIT`. However, if more than one data target is used you would need to rerun the *RDB Loader* with the successfully loaded target removed from the `targets` directory to retry loading the "failed" target.<br><br>**Note**: If the failure occurred at `analyze` stage, skip it with `--skip download,load` option (currently `analyze` is bound with `load` step and cannot be rerun separately).
 12 | Rerun the EmrEtlRunner with skipped *RDB Loader* step, by providing following option: `--skip rdb_load`.
