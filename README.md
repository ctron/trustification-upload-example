# Upload a bunch of SBOMs to trustification

## Check if you can do it?

```bash
podman run -ti --rm ghcr.io/trustification/trust:8b9f5f80ad9d773bc9de652bce5baa89d41dad99 /trust bombastic walker --help
```

This should print out the help for uploading SBOMs. If you see something explains how to use the command, you're good.

## Collecting required information

### Upload API endpoint

This is the Bombastic API endpoint.

### OIDC

We need:

* The issuer URL

  When using Keycloak/Red Hat SSO, that should be `https://<keycloak URL>/realms/<realm>`

* The client ID – that should be `walker`

* The client secret

  This can be extracted from the OpenShift cluster. Assuming it is deployed into the `trustification` namespace, you
  can get it using:

  ```bash
  oc -n trustification-prod get secret oidc-client-walker -o "jsonpath={.data.client-secret}" | base64 -d
  ```

  It also is possible to look up the `Secret` named `oidc-client-walker` in the OpenShift console and copy the secret
  manually. Just be careful to not copy the base64 encoded version. The "copy to clipboard" button will work.

## Preparing content

Assuming you have the content under `~/sboms`.

All SBOMs need to be `.bz2` compressed and located in the `~/sboms` folder. Inside the that directory, a metadata
file needs to be created. The full name of that file would be `~/sboms/metadata/metadata.json`. The content of that file
must be:

```json
{
  "keys": []
}
```

## Running

Set all required information to environment variables, which we will use during the next step.

```bash
export BOMBASTIC_URL=https://sbom.staging.trustification.dev/
export OIDC_PROVIDER_CLIENT_ID=walker
export OIDC_PROVIDER_CLIENT_SECRET=<id>
export OIDC_PROVIDER_ISSUER_URL=https://sso.staging.trustification.dev/realms/chicken
```

Then run the following command:

```bash
podman run -ti --rm -v $HOME/sboms:/data:z --env-host ghcr.io/trustification/trust:8b9f5f80ad9d773bc9de652bce5baa89d41dad99 /trust bombastic walker --v3-signatures --sink $BOMBASTIC_URL --source file:/data
```
