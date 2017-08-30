[**HOME**](Home) » [**SNOWPLOW SETUP GUIDE**](Setting-up-Snowplow) » Batch Pipeline Steps

*This page refers to Snowplow R91+*

*Click [here](Batch-pipeline-steps-r90) for the corresponding documentation for R90*

*Click [here](Batch-pipeline-steps-r89) for the corresponding documentation for R89*

*Click [here](Batch-pipeline-steps-r87) for the corresponding documentation for R87-R88*

*Click [here](Batch-pipeline-steps-r86) for the corresponding documentation for R86 and earlier*

## Dataflow diagram

[[/images/batch_pipeline_steps.png]]

## Recovery steps

The below table summarizes the actions to be taken at each particular step failure from the dataflow diagram above.

Failed step | Recovery actions
:---:|---
 1 | If no files have been moved yet (`raw:processing` [A] is empty), rerun the *EmrEtlRunner* as usual. If (on the other hand) some files have already been moved, rerun the *EmrEtlRunner* with `--skip staging` option to proceed with processing of those log files.
 2 | Rerun the *EmrEtlRunner* with `--skip staging` option.|
 3 | Rerun the *EmrEtlRunner* with `--skip staging` option.<br><br>**Note**: The `enriched:bad` [D] and `enriched:error` [E] could contain the files produced as a result of the step 3. Therefore rerunning the *EmrEtlRunner* could result in duplicated `bad`/`error` files. This could be significant if `elasticsearch` step [8-9] is engaged for examining `bad` data [D]. The outcome would be the same data timestamped with different time values by different EMR runs.
 4 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with either `--skip staging` option or with `--resume-from enrich`.
 5 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with either `--skip staging` option or with `--resume-from enrich`.
 6 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with either `--skip staging` option or with `--resume-from enrich`.
 7 | Delete `enriched:good` files [F] and rerun the *EmrEtlRunner* with either `--skip staging` option or with `--resume-from enrich`.<br><br>**Note**: The `enriched:bad` [D] and `shredded:bad` [H] could contain the files produced as a result of the step 3 and 6 respectively. Therefore rerunning the *EmrEtlRunner* could result in duplicated `bad` files. This could be significant if `elasticsearch` step (8-9) is engaged for examining `bad` data ([D],[H]). The outcome would be the same data timestamped with different time values by different EMR runs.
 8 | Delete `enriched:good` [F] and `shredded:good` [K]. Rerun the *EmrEtlRunner* with either `--skip staging` option or with `--resume-from enrich`.
 9 | If duplicated `bad` data is not critical rerun the *EmrEtlRunner* with `--skip staging,enrich,shred` option. If duplicated bad data **is** critical, *instructions to come ([#2593](https://github.com/snowplow/snowplow/issues/2593))*.<br><br>**WARNING**: In R90+ if you pass `--skip shred` to EmrEtlRunner then RDB Loader does not load unstructured events and contexts. This issue is to be resolved in R92.
 10 | If duplicated `bad` data is not critical rerun the *EmrEtlRunner* with `--skip staging,enrich,shred` option. If duplicated bad data **is** critical, *instructions to come ([#2593](https://github.com/snowplow/snowplow/issues/2593))*.<br><br>**WARNING**: In R90+ if you pass `--skip shred` to EmrEtlRunner then RDB Loader does not load unstructured events and contexts. This issue is to be resolved in R92.
 11 | Rerun the *EmrEtlRunner* with `--skip staging,enrich,shred,elasticsearch` option.<br><br>**WARNING**: In R90+ if you pass `--skip shred` to EmrEtlRunner then RDB Loader does not load unstructured events and contexts. This issue is to be resolved in R92.
 12 | The data load cannot result in partial load due to the use of `COMMIT`. However, if more than one data target is used you would need to rerun the *EmrEtlRunner* with the successfully loaded target removed from the `config.yml` configuration file to retry loading the "failed" target.<br><br>**Note**: If the failure occurred at `analyze` stage, you can skip it with `--resume-from archive_enriched` option. To analyze, resume with `--resume-from analyze`
 13 | Rerun the *EmrEtlRunner* with `--resume-from archive_enriched` option.
