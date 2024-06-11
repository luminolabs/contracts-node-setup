## Know before you go

Before I kick off, note that this is a relatively long tutorial! You should prepare to set aside an hour or two to get everything running. Here’s an itemized list of what we’re about to do:

1. Install dependencies
2. Build the source code
3. Generate and fund accounts and private keys
4. Configure your network
5. Deploy the L1 contracts
6. Initialize op-geth
7. Run op-geth
8. Run op-node
9. Get some Goerli ETH on your L2
10. Send some test transactions
11. Celebrate!

## Prerequisites

You’ll need the following software installed to follow this tutorial:

- [Git](https://git-scm.com/)
- [Go](https://go.dev/)
- [Node](https://nodejs.org/en/)
- [Pnpm](https://classic.yarnpkg.com/lang/en/docs/install/)
- [Foundry](https://github.com/foundry-rs/foundry#installation)
- [Make](https://linux.die.net/man/1/make)
- [jq](https://github.com/jqlang/jq)
- [direnv](https://direnv.net)

This tutorial was checked on:

| Software | Version    | Installation command(s) |
| -------- | ---------- | - |
| Ubuntu   | 20.04 LTS  | |
| git, curl, jq, and make | OS default | `sudo apt update` <br> `sudo apt install -y git curl make jq direnv` |
| Go       | 1.20       | `wget https://go.dev/dl/go1.20.linux-amd64.tar.gz` <br> `tar xvzf go1.20.linux-amd64.tar.gz` <br> `sudo cp go/bin/go /usr/bin/go` <br> `sudo mv go /usr/lib` <br> `echo export GOROOT=/usr/lib/go >> ~/.bashrc`
| Node     | 16.19.0    | `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh \| bash` <br> `export NVM_DIR="$HOME/.nvm"` <br> `[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm` <br> `[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion` <br> `nvm install 16`
| pnpm     | 8.5.6      | `npm install -g pnpm`
| Foundry  | 0.2.0      | `curl -L https://foundry.paradigm.xyz \| bash` <br> `source /root/.bashrc` <br> `foundryup`