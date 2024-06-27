# Obedience API
Documentation for the API of Obedience.

## Beta version
This API is in beta. Breaking changes to the API can (at this point) be made at any time. You'll be able to find up-to-date information here.

## Setup
To start, your extension needs to request the Obedience user's permission to connect to their account and access their data.

You can request the user's permission by sending them to `https://app.obedienceapp.com/home/extension-request?id=YOUR_ID&name=YOUR_EXTENSION_NAME&redirect=YOUR_REDIRECT_URL`,
where `YOUR_EXTENSION_ID` is a valid V4 UUID that is unique across the app (each extension must use a different ID for each Obedience user), `YOUR_EXTENSION_NAME` is the name of your extension that appears to the user, and `YOUR_REDIRECT_URL` is the URL where the user will be redirected after accepting the request.

When the user accepts the request, they will be redirected to `YOUR_REDIRECT_URL` with the following query parameters:
- `id`: same as `YOUR_EXTENSION_ID`,
- `secret`: a secret that your extension can use to access the API (for this specific Obedience user),
- `uid`: the user id of the Obedience user.

For example, if `YOUR_REDIRECT_URL` is `https://obedienceapp.com`, the user will be redirected to something like `https://obedienceapp.com?id=8ef02e0f-2428-4827-b411-ae887c1c599d&secret=212d7e4a-c1c8-477b-a123-351c15c4e32e&uid=0051lhgUrYO5HSpFa4Tt`.

Make sure the website you use as `YOUR_REDIRECT_URL` will save all three query parameters, as you will need these later.

## Webhooks
To set up a webhook, make a `POST` request to `https://app.obedienceapp.com/extensions/webhook` with the following query parameters:
- `extensionId`: same as `YOUR_EXTENSION_ID`,
- `secret`: the secret you got during setup

and a JSON body with the following format:
```
{
  'url': string,
  'habits': boolean,
  'rewards': boolean,
  'punishments': boolean
}
```

where `url` is your webhook endpoint, and `habits`, `rewards` and `punishments` indicate whether you want to receive webhook events for changes to the user's habits, rewards and punishments respectively.

Once you've enabled the webhook, and a relevant change is made, your webhook endpoint will receive a POST request with the following JSON body:
```
{
  "type": string,
  "extensionId": string,
  "secret": string,
  "before": ObjectData,
  "after": ObjectData
}
```
where `type` is either `habit`, `reward` or `punishment`, and `ObjectData` has the following format:
```
{
  "id": string,
  "owner": string,
  "partner": string,
  "name": string,
  "description": string,
  "amount": int,
  "cost": int (only for rewards!)
}
```
`id` is the ID of the object that changed, `owner` is the ID of the submissive who owns the habit and `partner` is the ID of the dominant partner (if applicable).

Note that for each webhook event emitted, there is a timeout of 10 seconds. Your webhook endpoint must therefore process the incoming POST request within 10 seconds. If you need more time, consider using a background job after the webhook event is received.
