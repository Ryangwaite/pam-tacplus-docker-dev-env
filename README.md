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
