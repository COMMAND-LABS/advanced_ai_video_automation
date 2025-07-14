# TLDR 

Configuring a custom "Dropbox OAuth2 Token" for n8n

## PART 1

- REFERENCE: https://www.dropbox.com/developers
- REFERENCE: https://developers.dropbox.com/oauth-guide (Contains detailed info)
- Go to https://www.dropbox.com/developers/apps
- Create app
  - Choose an API: Scoped access
  - Choose the type of access you need: Full Dropbox (App folder is probably better)
  - Name the OAuth Client Application: `aaiva_n8n`

## PART 2

- Go to: n8n Dashboard > Credentials tab
  - Create a new credential (OAuth2 API):
    - Edit credential Name: `Dropbox Custom OAuth2 Credential with refresh token`
    - OAuth Redirect URL: `https://oauth.n8n.cloud/oauth2/callback`
    - Grant Type: `Authorization Code`
    - Authorization URL: `https://www.dropbox.com/oauth2/authorize`
    - Access Token URL: `https://api.dropboxapi.com/oauth2/token`
    - Client ID (aka App key): find in your Dropbox App settings
    - Client Secret (aka App secret): find in your Dropbox App settings
    - Scope: leave empty or use `sharing.read sharing.write` if needed
    - Auth URI Query Parameters: `token_access_type=offline` (IMPORTANT!)
    - Authentication: Header

## PART 3

- Go to your app "Settings" tab in Dropbox
- Configure the Redirect URIs
  - `https://oauth.n8n.cloud/oauth2/callback` (Given to you by n8n)
- Configure the "Permissions" tab in Dropbox
- Submit changes

## PART 4

- Complete the OAuth 2.0 flow
- Using this OAuth2 token in n8n should include both the Dropbox OAuth2 access_token and refresh_token
- If the token stops working, you can come to the credentials tab in Nan and try refreshing or reconnecting it