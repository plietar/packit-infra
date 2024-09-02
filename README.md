# packit.dide.ic.ac.uk

## Preliminaries

You need Nix installed on your local dev machine, with flakes enabled. If you
don't have Nix installed, you can use SSH onto the server and use it to run the
commands. Obviously this doesn't work for initial provisioning.

TODO: maybe create some users on the server so we don't have to operate as root
for everything.

You can enable flakes by creating a `~/.config/nix/nix.conf` file:
```
experimental-features = nix-command flakes
```

## How do I deploy to the server?

```sh
nix run .#deploy
```

TODO: doing builds locally and then uploading them is pretty inefficient.
There's almost certainly a way to do the build remotely and not have to transfer
anything.

## How do I update outpack_server or Packit?

```
nix run .#update packit
nix run .#update outpack_server
```

The sources are defined in files at `packages/{packit,outpack_server}/source.json`.
The update script automatically fetches the latest revision from GitHub, and
updates the necessary hashes.

It also accepts a `--branch` argument which may be used to specify a branch
name. Otherwise the default branch of the repository will be used (usually main
or master).

## How do I build the deployment?

```sh
nix build
```

This is useful to check syntax and that everything builds, but you don't
typically need to do it. Deploying will do it implicitely.

## How do I visualize the effect of my changes?

```
nix run .#diff
```

This will download the system that is currently running on the server and
compare it with what would be deployed using [nix-diff](https://github.com/Gabriella439/nix-diff).

`nix-diff` will omit derivations' environment comparison if some dependencies
already differ. `nix run .#diff -- --environment` can help print all of these,
but is likely to be very verbose.

## How do I start a local VM?

```sh
nix run .#start-vm
```

This starts a local VM running in QEMU. Handy to check everything works as
expected before deploying.

It will forward port 443 of the VM onto port 8443, meaning you may visit
https://localhost:8443/ once the VM has started.

Nginx is configured using a self-signed certificate, which will cause some
browser warnings.

Packit is configured to use basic authentication, but no users exist by default.
In the VM console, run the following:

```
create-basic-user <instance> "admin@localhost.com" password
grant-role <instance> "admin@localhost.com" ADMIN
```

## How do I run the integration tests?

```sh
nix flake check
```

This doesn't test that much yet, just that the Packit API eventually comes up.
We should at least try to interact with the API a little.

## How do I add new SSH keys?

The keys are fetched from GitHub and committed into this repository as the
`authorized_keys` files. You can edit the `scripts/update-ssh-keys.sh` file to
update the list of users.

Afterwards run the following to fetch the new keys:
```sh
nix run .#update-ssh-keys
```

Carefully review the changes to avoid locking yourself out and re-deploy to the
server.

## How do I add a new Packit instance?

Edit `services.nix` by adding a new entry to the `services.multi-packit.instances`
attribute set. The name of the attribute will determine the URL of the
instance. Choose a new pair of unused port numbers for the outpack server and
for the packit API server.

A GitHub organisation and team needs to be specified. All members of this
organisation/team will be allowed to access the instance. The team can be
omitted or left blank, in which case any member of the organisation will have
access.

The Packit application on Github manually needs to be granted permission, by
one of the organisation's admins, to access the organisation. See [the GitHub
documentation][github-oauth-org]. Because we use a single OAuth app for all
instances, this only needs to be done once per org, even if uses by multiple
instances.

[github-oauth-org]: https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-your-membership-in-organizations/requesting-organization-approval-for-oauth-apps

The initial user needs to be granted the ADMIN role manually.

1. Log in to the instance with your GitHub account.
1. SSH onto the server.
1. Run `grant-role <instance> <username> ADMIN` where `<instance>` is the name
   of the instance and `<username>` is your GitHub username.
1. Log out and back in for the changes to take effect.

Afterwards permissions may be managed through the web UI.

## How do I update NixOS?

```
nix flake update
```

To switch to a new major version, you should edit the URL at the top of `flake.nix`.

## How do I provision a new machine?

See [`PROVISIONING.md`](PROVISIONING.md).
