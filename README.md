# pam-tacplus-dev-environment

## Overview

I've created this repo for the primary use case of reproducing and fixing this bug: https://github.com/kravietz/pam_tacplus/pull/166

Although it is likely to be useful beyond that 🙂

There's 2 docker containers:
- `pam_tacplus-client` - responsible for building, installing then running the `tacc` client
- `tac_plus` - runs a `tac_plus` server.

## Getting Started

1. Clone https://github.com/kravietz/pam_tacplus into the root of this repo.

```bash
$ git clone https://github.com/kravietz/pam_tacplus.git
```

2. Run docker-compose:
```bash
$ docker-compose up
```

3. In a new terminal obtain a bash terminal in `pam_tacplus-client` with:
```bash
docker-compose exec pam_tacplus-client bash
```
4. Build and install the module as follows:

```bash
$ autoreconf -i
$ ./configure --libdir=/lib
$ make
$ make install
```

For successive builds and installs run:
```bash
$ make clean all install
```
IDK why, but i seem to need to run the `clean` target between builds when configured this way. It's quick so not going to worry about fixing it.

> Note: I couldn't get it to work when the sources were installed into `/usr/local/lib/security/` with a corresponding `test-pam.conf` change. So i've opted for the install into `/lib/security` instead which works.

The pam_tacplus authentication logs can be found at `/var/log/syslog` in `pam_tacplus-client`

---

## Reproducing the bug

Run the following on a bash terminal in `pam_tacplus-client`:
```
# pamtester -v -I rhost=tac_plus test bigpuser authenticate acct_mgmt <<< default
pamtester: invoking pam_start(test, bigpuser, ...)
pamtester: performing operation - authenticate
Password: pamtester: successfully authenticated
pamtester: performing operation - acct_mgmt
pamtester: Permission denied
```
> Note: the `<<< default` heredoc supplies the password at the prompt.

`/var/log/syslog` in container `pam_tacplus-client` shows:
```
[...]
Dec 21 06:47:00 73932c90c2d3 pamtester: pam_sm_acct_mgmt: sent authorization request
Dec 21 06:47:00 73932c90c2d3 pamtester: tac_author_read_timeout: short reply body, read 28948 of 41656
Dec 21 06:47:00 73932c90c2d3 pamtester: PAM-tacplus: TACACS+ authorisation failed for [bigpuser]
```
> The last two logs show the problem. PAM-tacplus is processing a partially read Authorization-Reply packet instead of waiting and entirely reading it before processing it. The issue doesn't occur 100% of the time in this environment (maybe 70%?) so might need to try running the cmd again to hit it.

In comparison to a user with a small Authorization-Reply packet:
```
# pamtester -v -I rhost=tac_plus test smallpuser authenticate acct_mgmt <<< default
pamtester: invoking pam_start(test, smallpuser, ...)
pamtester: performing operation - authenticate
Password: pamtester: successfully authenticated
pamtester: performing operation - acct_mgmt
pamtester: account management done.
```
and syslog shows:
```
[...]
Dec 21 06:49:47 73932c90c2d3 pamtester: pam_sm_acct_mgmt: sent authorization request
Dec 21 06:49:47 73932c90c2d3 pamtester: Args cnt 0
Dec 21 06:49:47 73932c90c2d3 pamtester: pam_sm_acct_mgmt: user [smallpuser] successfully authorized
```

This is working as expected.

## Notes
- `docker-compose up` never rebuilds an image to to do that run `docker-compose build`

- To run just one of the containers (and remove the container afterwards):
`docker-compose run --rm tac_plus bash`

- To copy file out of the container run: `docker cp pam-tacplus-docker-dev-env_tac_plus_run_1379a5df0b19:/etc/tacacs+/tac_plus.conf .`

- The command for testing TACACS+ from within the pam_tacplus-client container is 
```bash
pamtester -v -I rhost=tac_plus test bigpuser authenticate acct_mgmt
```
