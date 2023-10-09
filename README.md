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

# Demo

```shell
cat > sfds.env <<EOF
# can be bare:
REPO=/path/to/git/repo
PORT=1337
INFO_KEY="$(tr -d -c '[A-Za-z0-9]' < /dev/urandom | head -c 96)"
BUILD_COMMAND="build_podman_container demo_parameter"
ALLOWED_REFS=main,staging
BUILD_BEARER_VALUES="$(tr -d -c '[A-Za-z0-9]' < /dev/urandom | head -c 96)"
# for each ref, there can be a list of services.
# $ref '=' $comma_separated [';' ..]..
# escape backslash, comma, and semicolon using backslash
SERVICES=main="demo.service,redis-production.service;staging=staging-app.service"
EOF
```
