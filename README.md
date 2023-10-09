# Small-Footprint Deployment Supervisor

When deploying applications from a single repo, it's sometimes practical to be able to monitor that deployment on-the-go from a browser.
This small-footprint deployment supervisor can be used to trigger CI/CD jobs and check service and build status/logs.

The supervisor is a single python file (python3). A simple worker is implemented in POSIX shell, relying on coreutils and git.
There's a consumer bash script for running builds, relying on curl.

# Technological Limitations/Details

- service monitoring makes use of systemd and journalctl
- flock (util-linux) is used to synchronize builds
- the shell script that uses the build service relies on bash arrays for correct quoting of arguments
- service monitoring assumes user-scoped services
- the source repository can be bare
- `SERVICES` is be used to specify services associated with a ref
  - the format is: `$ref '=' $comma_separated [';' ..]..`
  - escape backslash, comma, and semicolon using backslash

# Demo

```shell
# the default worker fetches 'origin' and wants the ref to be accessible from here:
git config remote.origin.fetch '+refs/heads/*:refs/remotes/origin/heads/*'

cat > example/env <<EOF
REPO=$(pwd)
PORT=1337
INFO_KEY=$(tr -d -c '[A-Za-z0-9]' < /dev/urandom | head -c 96)
BUILD_COMMAND=$(pwd)/example/build checksum
ALLOWED_REFS=main,staging
BUILD_BEARER_VALUES=$(tr -d -c '[A-Za-z0-9]' < /dev/urandom | head -c 96)
SERVICES=main=demo.service,redis-production.service;staging=staging-app.service
EOF

while read -r line; do
	export -- "$line"
done < example/env

mkdir -p example/deployments

( cd example/deployments ; ../../sfds_supervisor ; ) &

# putting the FORCE_RETRY=1 here to make testing easier
FORCE_RETRY=1 AUTH_BEARER="$(echo "$BUILD_BEARER_VALUES" | cut -f1 -d,)" ./sfds_use http://localhost:1337 main
```
