# Veramo Agent Ansible Deploy

This Ansible playbook deploys a single Veramo based agent for use in the
Eduwallet Proof-of-Concept setup. It clones and configures the
[veramo-agent](https://github.com/muisit/veramo-agent) using the configurations
defined in the group_vars settings.

The end result is a docker-compose configuration containing 3 docker containers:

- a front-end facing Traefik container taking care of SSL certificates
- an agent based on the Sphereon SSI-SDK using the Veramo core Typescript framework
- a Postgres database

The Ansible playbook creates the docker-compose file, builds the docker containers, provides a convenient start/stop script and runs the latter to start-up the containers.

## Concepts

### DIDs

A list of Decentralized Identifiers found in group\_vars all. One for each
Installation (see below). We use did:web, so we provide a domain. See [the
spec](https://w3c-ccg.github.io/did-method-web/) for how to define ports and
paths and such. Don't add fragments (#something), though as these are added by
ansible templates where needed. 

### Credential Configurations

A list of credential configurations found in group\_vars all. One for each
credential configuration that an issuer can offer credentials for.

### Installations

A list found in group_vars all. This contains the configurations for all issuers.
For each issuer, a *configuration* and *metadata* file is generated in the conf
directory as json file.

The agent (in a docker container) then picks up these config files on boot, and
provides API endpoints and well-known endpoints for each issuer using these files.

We provide a single credential, as set up in Credential Configurations, that this issuer
can issue credentials from.

## Invocation

Running the playbook can be done using the convenience script `run`. This will invoke the `ansible-playbook` command with the `eduwallet` environment inventory and the `agent` playbook and configure the Ansible Vault secrets. 

You can supply a different inventory directory as the first argument, for example: `run mysettings`. In this case, create a directory called `mysettings` under the `environments` directory containing an `inventory` file listing the host inventory.

Group variables are retrieved from the `group_vars` directory and are common to all environments. There is currently one group: `all`.

## Secrets

Secrets are encoded using Ansible Vault. The Vault ID is stored in the local `ansible_vault_id` file. Share this file content to get access to the secrets encoded in the environment variables. Do not store this file in a public repository!

This repository contains a convenience script to encrypt and decrypt secrets using Ansible Vault:

- invoke `./encrypt <secret>` to encrypt the secret. The result can be included in the environment variables
- invoke `./encrypt` to encrypt a secret pasted or piped to STDIN. The result can be included in the environment variables
- invoke `./encrypt -d <encrypted secret>` to decrypt a secret as copied from the environment variables to see what is actually encoded.

Use the `-f` option to encode or decode whole files.
If no `<secret>` is provided, it is read from STDIN.

Examples:

- `./encrypt TestMe`
- `echo "secret" | ./encrypt`
- `cat encoded_secret | ./encrypt -d`

An encoded secret typically looks like:

```text
$ANSIBLE_VAULT;1.1;AES256
6631626537613461363362326330316338353563626130343438396436353733613862393639623132353162313965343439326233
31666139323365336135330a3737353134363533646536333739646566303633396365323330663162613933633835333730316364
62356233383266613763376531613137363333613766640a3563643432393863373837386437383961633765663930303331656337
623165
```

Include this in your environment settings using proper indentation:

```text
secretname: !vault |
            $ANSIBLE_VAULT;1.1;AES256
            6631626537613461363362326330316338353563626130343438396436353733613862393639623132353162313965
            34343932623331666139323365336135330a3737353134363533646536333739646566303633396365323330663162
            61393363383533373031636462356233383266613763376531613137363333613766640a3563643432393863373837
            386437383961633765663930303331656337623165
```
