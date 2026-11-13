# ns8-opensign

Module id: `opensign`. Images are pinned in `build-images.sh` and updated through reviewed changes.

## Install

```bash
add-module ghcr.io/geniusdynamics/opensign:latest 1
```

The command returns the instance id, for example `opensign1`. Use the id NS8 returns.

## Configure

```bash
api-cli run configure-module --agent module/opensign1 --data - <<'EOF'
{
  "host": "opensign.example.org",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

| Parameter | Required | Type | Description |
|---|---|---|---|
| `host` | yes | string | Host name for the application, like 'opensign.domain.org' |
| `lets_encrypt` | yes | boolean | Request a valid Let's Encrypt certificate. |
| `http2https` | yes | boolean | Redirect all the HTTP requests to HTTPS |

## Configuration output

`get-configuration` returns: `host`, `lets_encrypt`, `http2https`.

## Images

| Image | Version |
|---|---|
| `docker.io/mongo` | `9.0.8` |
| `docker.io/opensign/opensign` | `main` |
| `docker.io/opensign/opensignserver` | `main` |

## Network and access

- Reserves 2 TCP port(s) on the node. The web application is published through Traefik on the configured host name.
- Runs rootless.
- Declares these NS8 authorizations: `traefik@node:routeadm`.

## Backup

The following paths are included in the NS8 backup (`imageroot/etc/state-include.conf`):

- `state/opensign.archive`
- `state/mongo.env`
- `state/backend.env`
- `state/backend-db.env`
- `state/pfx.env`
- `state/frontend.env`
- `state/storage.env`
- `state/email.env`
- `volumes/mongo-app`
- `volumes/mongo-configdb`
- `volumes/./apps/OpenSign`
- `volumes/./apps/OpenSignServer`

## Services

- `mongo-app.service`
- `opensign-client.service`
- `opensign-server.service`
- `opensign.service`

## Get the configuration

```bash
api-cli run get-configuration --agent module/opensign1
```

## Update and uninstall

```bash
api-cli run update-module --data '{"module_url":"ghcr.io/geniusdynamics/opensign:latest","instances":["opensign1"],"force":true}'
remove-module --no-preserve opensign1
```

## Testing

Continuous integration:

- `Publish images` builds the module image on every push.
- `Test module` installs the image on a QEMU NS8 node and runs the Robot Framework suite in `tests/`. It runs after `Publish images` succeeds on `dev` and on `main`.
- `Build API JSON Schema docs` and the security scan run alongside.

The suite installs the module, configures a host name, checks that the web application answers through Traefik, takes UI screenshots and removes the module.

Run it locally with the NS8 standard infrastructure described in the [ns8-github-actions README](https://github.com/NethServer/ns8-github-actions/blob/v1/README.md#running-tests-locally).

## Open items

Found by the NS8 module standards check. These must be resolved before a stable release:

- The module description is still the template text: write a description of at least 60 characters in ui/public/metadata.json
- metadata: categories empty (use standard categories)
- metadata: description.en needs >= 60 real chars (+ it translation)
- opensign-client.service: uses %S/state, kickstart uses %E/state
- opensign-server.service: uses %S/state, kickstart uses %E/state
- README still has the template boilerplate
- opensign-client.service: %S/state changed to %E/state (review the bind mounts that remain)
- opensign-server.service: %S/state changed to %E/state (review the bind mounts that remain)
- MONGO_IMAGE is used by the module but no image in the label provides it: leftover from the template?
- OPENSIGNSERVER_IMAGE is used by the module but no image in the label provides it: leftover from the template?
- OPENSIGN_IMAGE is used by the module but no image in the label provides it: leftover from the template?

## Upstream

Application documentation: https://docs.opensignlabs.com/

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
