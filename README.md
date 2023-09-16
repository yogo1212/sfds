# Small-Footprint Deployment Supervisor



The supervisor is a single python file - most people have got python on their servers.
It can be used for simple CI/CD tasks but also offers a minimalistic frontend to monitor builds and services.
The shell script for performing builds should be usable from almost anywhere.

Here are a few technological limitations:

- service monitoring makes use of systemd and journalctl
- flock (util-linux) is used to synchronize builds
- the shell script that uses the build service relies on bash arrays for correct quoting of arguments
- service monitoring assumes user-scoped services - try to avoid running builds and services as root!

# Demo

```shell
cat > sfds.env <<EOF
# can be bare:
REPO=/path/to/git/repo
PORT=1337
INFO_KEY=$(tr -d -c '[A-Za-z0-9]' < /dev/urandom | head -c 96)
BUILD_COMMAND=build_podman_container demo_parameter
ALLOWED_REFS=main,staging
BUILD_BEARER_VALUES=$(tr -d -c '[A-Za-z0-9]' < /dev/urandom | head -c 96)
# for each ref, there can be a list of services.
# $ref '=' $comma_separated [';' ..]..
# escape backslash, comma, and semicolon using backslash
SERVICES=main=demo.service,redis-production.service;staging=staging-app.service
EOF
```
