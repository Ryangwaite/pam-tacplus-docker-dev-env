# pam-tacplus-dev-environment



## Overview

There's 2 docker containers:
- `pam_tacplus-client` - responsible for building, installing then running the `tacc` client
- `tac_plus` - runs a `tac_plus` server.

I'll build the dockerfiles with these best practices in mind: https://github.com/hexops/dockerfile

1. Clone https://github.com/kravietz/pam_tacplus into the root of this repo.

```bash
git clone https://github.com/kravietz/pam_tacplus.git
```

2. Run docker-compose:
```bash
docker-compose up
```

## Notes
- `docker-compose up` never rebuilds an image to to do that run `docker-compose build`

- To run just one of the containers (and remove the container afterwards):
`docker-compose run --rm tac_plus bash`

- To copy file out of the container run: `docker cp pam-tacplus-docker-dev-env_tac_plus_run_1379a5df0b19:/etc/tacacs+/tac_plus.conf .`

- To get a terminal to an already docker-compose up'd container run `docker-compose exec tac_plus bash`
