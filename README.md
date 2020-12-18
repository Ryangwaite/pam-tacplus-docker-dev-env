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

TODO: add notes on running `make install`

## Reproducing the bug

Run the following on a bash terminal in `pam_tacplus-client`:
```
# pamtester -v -I rhost=tac_plus test bigpuser authenticate acct_mgmt <<< default
pamtester: invoking pam_start(test, bigpuser, ...)
pamtester: performing operation - authenticate
pamtester: successfully authenticated
pamtester: performing operation - acct_mgmt
pamtester: Permission denied
```
> Note: the `<<< default` heredoc supplies the password at the prompt.

`/var/log/auth.log` in container `pam_tacplus-client` shows:
```
Dec 18 09:37:52 74c69bb871fa PAM-tacplus[271]: 1 servers defined
Dec 18 09:37:52 74c69bb871fa PAM-tacplus[271]: server[0] { addr=172.20.0.2:49, key='********' }
Dec 18 09:37:52 74c69bb871fa PAM-tacplus[271]: tac_service=''
Dec 18 09:37:52 74c69bb871fa PAM-tacplus[271]: tac_protocol=''
Dec 18 09:37:52 74c69bb871fa PAM-tacplus[271]: tac_prompt=''
Dec 18 09:37:52 74c69bb871fa PAM-tacplus[271]: tac_login=''
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_authenticate: called (pam_tacplus v1.3.8)
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_authenticate: user [bigpuser] obtained
Dec 18 09:37:52 74c69bb871fa pamtester[271]: tacacs_get_password: called
Dec 18 09:37:52 74c69bb871fa pamtester[271]: tacacs_get_password: obtained password
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_authenticate: password obtained
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_authenticate: tty [unknown] obtained
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_authenticate: rhost [tac_plus] obtained
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_authenticate: trying srv 0
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_authenticate: active srv 0
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_authenticate: exit with pam status: 0
Dec 18 09:37:52 74c69bb871fa PAM-tacplus[271]: 1 servers defined
Dec 18 09:37:52 74c69bb871fa PAM-tacplus[271]: server[0] { addr=172.20.0.2:49, key='********' }
Dec 18 09:37:52 74c69bb871fa PAM-tacplus[271]: tac_service='ppp'
Dec 18 09:37:52 74c69bb871fa PAM-tacplus[271]: tac_protocol='ip'
Dec 18 09:37:52 74c69bb871fa PAM-tacplus[271]: tac_prompt=''
Dec 18 09:37:52 74c69bb871fa PAM-tacplus[271]: tac_login=''
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_acct_mgmt: called (pam_tacplus v1.3.8)
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_acct_mgmt: username obtained [bigpuser]
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_acct_mgmt: tty obtained [unknown]
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_acct_mgmt: rhost obtained [tac_plus]
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_acct_mgmt: active server is [172.20.0.2:49]
Dec 18 09:37:52 74c69bb871fa pamtester[271]: pam_sm_acct_mgmt: sent authorization request
Dec 18 09:37:52 74c69bb871fa pamtester[271]: tac_author_read: short reply body, read 14468 of 41656: Operation now in progress
Dec 18 09:37:52 74c69bb871fa PAM-tacplus[271]: TACACS+ authorisation failed for [bigpuser]

```
> The last two logs show the problem. PAM-tacplus is processing a partially read Authorization-Reply packet instead of waiting and entirely reading it before processing it.

In comparison to a user with a small Authorization-Reply packet:
```
# pamtester -v -I rhost=tac_plus test smallpuser authenticate acct_mgmt <<< default
pamtester: invoking pam_start(test, smallpuser, ...)
pamtester: performing operation - authenticate
pamtester: successfully authenticated
pamtester: performing operation - acct_mgmt
pamtester: account management done.
```
and syslog shows:
```
Dec 18 09:39:37 74c69bb871fa PAM-tacplus[272]: 1 servers defined
Dec 18 09:39:37 74c69bb871fa PAM-tacplus[272]: server[0] { addr=172.20.0.2:49, key='********' }
Dec 18 09:39:37 74c69bb871fa PAM-tacplus[272]: tac_service=''
Dec 18 09:39:37 74c69bb871fa PAM-tacplus[272]: tac_protocol=''
Dec 18 09:39:37 74c69bb871fa PAM-tacplus[272]: tac_prompt=''
Dec 18 09:39:37 74c69bb871fa PAM-tacplus[272]: tac_login=''
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_authenticate: called (pam_tacplus v1.3.8)
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_authenticate: user [smallpuser] obtained
Dec 18 09:39:37 74c69bb871fa pamtester[272]: tacacs_get_password: called
Dec 18 09:39:37 74c69bb871fa pamtester[272]: tacacs_get_password: obtained password
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_authenticate: password obtained
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_authenticate: tty [unknown] obtained
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_authenticate: rhost [tac_plus] obtained
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_authenticate: trying srv 0
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_authenticate: active srv 0
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_authenticate: exit with pam status: 0
Dec 18 09:39:37 74c69bb871fa PAM-tacplus[272]: 1 servers defined
Dec 18 09:39:37 74c69bb871fa PAM-tacplus[272]: server[0] { addr=172.20.0.2:49, key='********' }
Dec 18 09:39:37 74c69bb871fa PAM-tacplus[272]: tac_service='ppp'
Dec 18 09:39:37 74c69bb871fa PAM-tacplus[272]: tac_protocol='ip'
Dec 18 09:39:37 74c69bb871fa PAM-tacplus[272]: tac_prompt=''
Dec 18 09:39:37 74c69bb871fa PAM-tacplus[272]: tac_login=''
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_acct_mgmt: called (pam_tacplus v1.3.8)
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_acct_mgmt: username obtained [smallpuser]
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_acct_mgmt: tty obtained [unknown]
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_acct_mgmt: rhost obtained [tac_plus]
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_acct_mgmt: active server is [172.20.0.2:49]
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_acct_mgmt: sent authorization request
Dec 18 09:39:37 74c69bb871fa pamtester[272]: pam_sm_acct_mgmt: user [smallpuser] successfully authorized
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
