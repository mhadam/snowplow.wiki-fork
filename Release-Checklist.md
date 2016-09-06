All the things to tick off when doing a new release:

0. **Complete?** - make sure all tickets completed
1. **CHANGELOG** - updated, including version and date? Remove any `DONE` or `WIP` notes.
2. **Documentation** - any urgent documentation changes release at the same time as the new version; any lower priority changes,  file a ticket in the snowplow repo
3. **Configuration file templates** - easy to forget. Have these been updated?
4. **Hosted assets** - any new ones, add to the hosted assets page. Also, remove any `-TEST-rX` suffixes on the filenames of the new hosted asset versions
5. **References to hosted assets** - e.g. in HTML files or in config files. Make sure to update these
6. **Cut the release** - merge `master` -> `develop`, merge `feature/new-release` -> `develop`, all okay? Then merge `develop` -> `master` and `push`
7. **Tag** - always tag like so: `git tag -a '0.6.2' -m 'Version 0.6.2'`. Then `git push --tags`
8. **Issues** - close any issues fixed in this release
9. **Roadmap** - update the overall roadmap (delete the just-released release)
10. **Delete feature branch** - `git branch -D feature/whatever` and `git push origin :feature/whatever`
11. **Blog post** - always write a blog post
12. **Add release** - in GitHub
13. **Update the version matrix** - in Google Spreadsheets
14. **Ticket to Ihor to update upgrade guide**
14. **Tweet** - put `#opensource` in the tweet
15. **Thread on Discourse** - final step