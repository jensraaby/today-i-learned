# Git tidbits

## Diff all the changes

If you want to double check what's changed before staging files, just run `git diff`.

I use `git diff path/to/file` all the time, but the aggregated version is sometimes more useful:

```
$ git diff
diff --git a/git/misc-stuff.md b/git/misc-stuff.md
index 740eeda..6dc156e 100644
--- a/git/misc-stuff.md
+++ b/git/misc-stuff.md
@@ -1,9 +1,12 @@
 # Git tidbits

-## Diff all the stuff
+## Diff all the changes

-If you want to double check what's changed before staging files, just run `git diff`
+If you want to double check what's changed before staging files, just run `git diff`.
+
+I use `git diff path/to/file` all the time, but the aggregated version is sometimes more useful:

 ```

+
 ```
\ No newline at end of file
diff --git a/git/no-feature-branches.md b/git/no-feature-branches.md
index 1d07b45..0eac95f 100644
--- a/git/no-feature-branches.md
+++ b/git/no-feature-branches.md
@@ -2,3 +2,4 @@

 In my team we have a strict workflow of committing to master. The software should always be releasable, and we don't want any branches to stick around for ages while the master branch moves on.

+This requires some care if you are working on a new feature that cannot be deployed yet, but it is generally easy to introduce a toggle so you can temporarily disable the functionality without deleting (or heaven forbid, *commenting out*) any code.

```