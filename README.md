# pam-tacplus-dev-environment

## Overview

There's 2 docker containers:
- `pam_tacplus-client` - responsible for building, installing then running the `tacc` client
- `tac_plus` - runs a `tac_plus` server.

## Getting Started

1. Clone https://github.com/kravietz/pam_tacplus into the root of this repo.

```bash
git clone https://github.com/kravietz/pam_tacplus.git
```

2. Run docker-compose:
```bash
docker-compose up
```

3. In a new terminal obtain a bash terminal in `pam_tacplus-client` with:
```bash
docker-compose exec pam_tacplus-client bash
```

## Reproducing the bug

Run the following on a bash terminal in `pam_tacplus-client`:
```bash
$ pamtester -v -I rhost=tac_plus test bigpuser authenticate acct_mgmt
pamtester: invoking pam_start(test, bigpuser, ...)
pamtester: performing operation - authenticate

pamtester: successfully authenticated
pamtester: performing operation - acct_mgmt
pamtester: Permission denied
```
> Note: that blank line is when pam_tacplus authentication pauses as it awaits user input. Type _default_ followed by enter to continue.

syslog shows:
```bash
TODO
```
In comparison to a user with a small Authorization-Reply packet:
```bash
$ pamtester -v -I rhost=tac_plus test smallpuser authenticate acct_mgmt
pamtester: invoking pam_start(test, smallpuser, ...)
pamtester: performing operation - authenticate

pamtester: successfully authenticated
pamtester: performing operation - acct_mgmt
pamtester: account management done.
```
and syslog shows:
```bash
TODO
```

## Notes
- `docker-compose up` never rebuilds an image to to do that run `docker-compose build`

- To run just one of the containers (and remove the container afterwards):
`docker-compose run --rm tac_plus bash`

- To copy file out of the container run: `docker cp pam-tacplus-docker-dev-env_tac_plus_run_1379a5df0b19:/etc/tacacs+/tac_plus.conf .`

- The command for testing TACACS+ from within the pam_tacplus-client container is 
```bash
pamtester -v -I rhost=tac_plus test bigpuser authenticate acct_mgmt
```
