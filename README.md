# FreeBSD relayd

## Building

```
env LIBCRYPTO=... LIBSSL=... LIBTLS=... LOCALBASE=... OPENSSLINCDIR=... PREFIX=... ./configure
make
```

## Compatibility

The latest tested LibreSSL version is 3.7.2.

## Style

1. Use `freebsd-relayd: ` prefix in commit message.

2. Use `#if __FreeBSD__`, `#endif` to make FreeBSD specific blocks in
   source files.

3. Use `# BEGIN FreeBSD-relayd`, `# END FreeBSD-relayd` comments to make
   FreeBSD specific blocks in makefiles and configuration files

## FreeBSD relayd release process

### Publishing a new branch

1. Create a new branch called `release`.

   ```sh
   git checkout -b release
   ```

2. Add the OpenBSD repository as a remote.

   ```sh
   git remote add openbsd git@github.com:openbsd/src.git
   ```

3. Pull the master branch from the `openbsd` remote and track it as `openbsdmaster`.

   ```sh
   git fetch openbsd
   git checkout -b openbsdmaster --track openbsd/master
   ```

4. Find the details needed for the final branch name.

   The name of the final branch should be in the form of the following scheme:

   ```
   OSMAJOR.OSMINOR.LATEST_COMMIT_DATE
   ```

   `OSMAJOR` and `OSMINOR` are major and minor versions set in the OpenBSD
   source tree in the master branch. `LATEST_COMMIT_DATE` is the commit date
   of the latest OpenBSD commit in the master branch (e.g., `2014.08.10`).

   The `OSMAJOR` and `OSMINOR` values can be found in `share/mk/sys.mk`:

   ```sh
   grep "OSM\(AJOR\|INOR\)=" share/mk/sys.mk
   ```

   `LATEST_COMMIT_DATE` can be obtained with the following command:

   ```sh
   git show --quiet HEAD | grep "Date:"
   ```

5. Rename the `release` branch to `OSMAJOR.OSMINOR.LATEST_COMMIT_DATE`.

   ```sh
   git branch -m "release" "OSMAJOR.OSMINOR.LATEST_COMMIT_DATE"
   ```

6. Rebase and merge the final branch (`OSMAJOR.OSMINOR.LATEST_COMMIT_DATE`) on
   top the `openbsdmaster` branch.

   ```
   git rebase --merge openbsdmaster
   ```

   Resolve conflicts if needed.

7. Check the build process.

   ```
   ./configure
   make
   ```

8. Test relayd.

   Currently, the testing procedure is not automated. Testing should include the
   following steps:

   1. Create a configuration file for relayd.
   2. Check the configuration file's syntax: `relayd -n -v -f <config>`.
   3. Run relayd: `relayd -d -v -f <config>`.

9. Publish the final branch to the `freebsd-relayd` repository.

   ```sh
   git push --set-upstream freebsd-relayd "OSMAJOR.OSMINOR.LATEST_COMMIT_DATE"
   ```

### Publish a release from a version branch

1. Make sure that the local repository is clean. Best way is to either work on
   a fresh clone or by running `git clean -fdx`, which removes all untracked
   files and directories.

2. Tag a release. The tag should follow this format:
   `OSMAJOR.OSMINOR.LATEST_COMMIT_DATE-pPATCHLEVEL`, where `PATCHLEVEL` is the
   next tag number on the version branch (starting with 0),
   e.g., `7.3.2023.05.09-p0`.

   ```sh
   git tag OSMAJOR.OSMINOR.LATEST_COMMIT_DATE-pPATCHLEVEL
   ```

3. Create `relayd-OSMAJOR.OSMINOR.LATEST_COMMIT_DATE-pPATCHLEVEL.tar.gz`
   with `release-relayd.sh`.

   ```sh
   ./release-relayd.sh OSMAJOR.OSMINOR.LATEST_COMMIT_DATE-pPATCHLEVEL
   ```

   Make sure that the archive actually contains all the necessary files to
   build relayd.

4. Publish the tag.

   ```sh
   git push --tags
   ```

5. Create a new release on GitHub based on the newly created tag and attach the
   `relayd-OSMAJOR.OSMINOR.LATEST_COMMIT_DATE-pPATCHLEVEL.tar.gz` archive to
   the release page.
