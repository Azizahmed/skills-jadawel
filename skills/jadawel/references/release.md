# Releasing Jadawel

## The shape

```
push to main (code92-dev/jadawel_cranl)
  → "Publish all-in-one image" workflow (GitHub runner)
      → ghcr.io/code92-dev/jadawel_cranl:<tag>  (also :latest)
          → root Dockerfile pins that image BY DIGEST
              → owner redeploys + reloads the CranL app "jadawel-org" (Saudi region)
                  → https://app.jadawl.site
```

CranL cannot build the monorepo. The Nuxt build needs more than its 4 GB of
memory, so CranL only pulls a finished image. As a result:

- **Pushing code deploys nothing.** A deploy happens only when the pin changes
  and your owner redeploys.
- **The pin on `main` is not proof of what is live.** What is live is whatever
  your owner last redeployed. If it matters, ask, then confirm against the
  behaviour of production.
- **CranL's "done" is not proof.** Once, CranL reported a deploy as done while
  the old workers kept running: an edit to the digest alone does not invalidate
  CranL's build cache. A redeploy needs a **Reload** afterwards, and then a check
  of real behaviour.

Your owner presses the CranL buttons. No CranL credentials exist outside the
dashboard.

## Steps

Get approval before step 2 and again before step 5.

1. **Check readiness.** The commits are on `origin/main` and the *Jadawel CI*
   run for that commit is green. Check with
   `gh run list -R code92-dev/jadawel_cranl --workflow jadawel-ci.yml --commit <sha>`.
   CI takes about 33 minutes across seven jobs.
2. **Choose the tag.** Use `2.3.<n>-<slug>`, where `<n>` is one more than the
   last release and `<slug>` names the change in kebab case. The last release is
   in the comment block above `ARG JADAWEL_IMAGE` in the root `Dockerfile`
   ("Published … tag 2.3.17-security-audit").
3. **Publish.**
   `gh workflow run publish-image.yml -R code92-dev/jadawel_cranl --ref main -f tag=<tag>`,
   then follow it with `gh run watch`. A warm run takes about 6 minutes and a
   cold one 30 to 60. Republishing an existing tag fails unless
   `-f overwrite=true` is set, which needs approval.
4. **Get the digest.**
   `docker buildx imagetools inspect ghcr.io/code92-dev/jadawel_cranl:<tag>`
   and take the top-level `Digest: sha256:…`.
5. **Pin it.** In the root `Dockerfile`, set
   `ARG JADAWEL_IMAGE=ghcr.io/code92-dev/jadawel_cranl@sha256:<digest>` and
   rewrite the release note above it, keeping the previous pin as
   "Previous deployment pin". The note covers: the date, the commit and tag, what
   changed, the tests run, whether there is a **migration**, and whether there are
   **environment changes**. Commit as `chore(deploy): pin <tag>` and push.
6. **Hand over to your owner.** Tell them: "Redeploy jadawel-org in CranL, then
   Reload." Add any migration or environment change they must make first.
7. **Verify** once they confirm. Done when every check passes:
   - `GET https://app.jadawl.site/api/_health/` returns 200.
   - Anonymous `GET /api/billing/admin/plans/` and `/api/organizations/admin/`
     return an authentication error, not `URL_NOT_FOUND`.
   - The feature this release shipped behaves as intended on production.

   Report each check with its result. If any check fails, report it and stop.
   Rolling back means re-pinning the previous digest, and that needs approval.

## Repository facts

- There is one remote: `origin` is `https://github.com/code92-dev/jadawel_cranl`.
  It serves for both development and deployment. Any note in the repository
  about a separate `cranl` remote or `Azizahmed/…` repositories is out of date.
- History is linear: rebase, never merge. Commit subjects are Conventional
  Commits (`feat(dashboard):`, `fix(mcp):`, `chore(deploy):`).
- Before a push, two fork-specific checks must pass: Arabic locale parity
  (`yarn locale:check` in `web-frontend/`) and fork hygiene
  (`pytest tests/arabase -q` in `backend/`).
- Coolify and `jadawel.azoz.cloud` are retired, and nothing deploys there.
