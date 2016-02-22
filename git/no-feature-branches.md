# No feature branches

In my team we have a strict workflow of committing to master. The software should always be releasable, and we don't want any branches to stick around for ages while the master branch moves on.

This requires some care if you are working on a new feature that cannot be deployed yet, but it is generally easy to introduce a toggle so you can temporarily disable the functionality without deleting (or heaven forbid, *commenting out*) any code.
