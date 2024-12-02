# PDS on fly.io

For running a [PDS](https://github.com/bluesky-social/pds) on [fly.io](https://fly.io) so you can host your own data, but not have to maintain your own servers.

## Instructions

This assumes you know your way around a command line and BlueSky, generally.

1. Pre-reqs
    1. Sign up for fly.io and install their CLI
    1. Find or buy a domain name to sacrifice (e.g., example.com)
    1. Find an e-mail or e-mail service to use (a free gmail works just fine)
1. `git clone` this repo
1. Modify `fly.toml` with your own `app` and `primary_region` values. (You can try to use mine, but probably won't work very well.)
1. Set environmental variables.
    1. $ `cp pds.env.example pds.env`
    1. Fill in the requisite values in pds.env
1. Launch the app
    1. $ `fly launch` <== create the app (make sure you picked a cool name!)
    1. $ `cat pds.env | fly secrets import` <== move the secrets over to the PDS host
    1. $ `fly ips list` <== you'll need this to get the IPv4 xxx.xxx.xxx.xxx for your DNS
1. Sacrifice your domain name (set the A-records in your DNS, per [instruction](https://github.com/bluesky-social/pds))
1. Add certs for your domain name
    1. $ `fly certs add example.com` <== root domain
    1. $ `fly certs add "*.example.com"` <== wildcard
    1. Add CNAME record to DNS per instructions provided in the command line
    1. Wait for it (and read [the docs](https://fly.io/docs/networking/custom-domain/) while you wait in case something goes wrong, which it did for me because Cloudflare likes to be overly-helpful sometimes)
1. Load `example.com/xrpc/_health` in a browser to see if it loads. Should show you a version number in JSON format. (Woo!)
1. [Generate an invitation](https://github.com/bluesky-social/pds/blob/main/pdsadmin/create-invite-code.sh)
    1. Use the curl command [below](#generate-an-invitation)
    1. Hold onto the output for the next step
1. Sign up for a new account using your custom Server in the BlueSky app
    1. Change your `Hosting Provider` to `Custom`
    1. Enter your PDS's domain name (e.g., `example.com`)
    1. Enter your invite code

You now have an account on your own PDS. How about that?

### Generate an Invitation

Replace the `${VALUE}`s (exclusive of the `${}`) from your pds.env

```sh
curl \
  --fail \
  --silent \
  --show-error \
  --request POST \
  --user "admin:${PDS_ADMIN_PASSWORD}" \
  --header "Content-Type: application/json" \
  --data '{"useCount": 1}' \
  "https://${PDS_HOSTNAME}/xrpc/com.atproto.server.createInviteCode" | jq --raw-output '.code'
```

## But, why?

Because I think all of this is [very neat indeed](https://michaelgruen.com/d/2024/human-dns/).
