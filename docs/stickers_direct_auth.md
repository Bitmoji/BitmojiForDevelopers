# Bitmoji Direct - Standalone Stickers

The Standalone stickers API offers an easy way to create personalized experiences featuring Bitmoji. This feature allows the creation of URLs using an avatar and stickers from the Bitmoji collection.

## User avatar

The base to create all custom URLs is the user avatar. This is an identifier that represents a user uniquely. This ID is unique per partner to guarantee privacy and separation. Currently, the only method to obtain a user's avatar is authenticating through Login Kit, the authentication SDK for Snap Kit.

### Fetching the avatar through Bitmoji Direct API

This guide assumes familiarity with Bitmoji Direct Auth. You'll need to configure your app in the [developer portal](https://snapkit.com/docs/developer-portal).

Once you are able to get the access token for a user, you can fetch their avatar_id using the `https://bitmoji.api.snapchat.com/direct/avatar` endpoint. An example using `curl`:

```bash
$ TOKEN=<your_access_token>
$ curl -X POST https://bitmoji.api.snapchat.com/direct/avatar \
-H "Authorization: Bearer $TOKEN"

{"id":"AUxuZkpWrWuY1hVhLWz6kFNO~bJ8mg"}
```

### Selfie URL

An image able to represent a user in a profile is called a selfie. It is straightforward to create a selfie given an avatar ID. The format of the URL is

```
https://sdk.bitmoji.com/me/sticker/<avatar_id>
```

so the example avatar id above combined with the selfie URL would be:

```
https://sdk.bitmoji.com/me/sticker/AWVscGhvWez65Dau7FgShF5VPT5C7A
```

![](https://sdk.bitmoji.com/me/sticker/AWVscGhvWez65Dau7FgShF5VPT5C7A)


## Custom stickers

This section will describe how to expand the selfie idea to allow the creation of customized stickers.

### Getting the Sticker IDs for your app

The first step is to decide which are the Bitmoji images you want to use in your app. You can work with the Bitmoji team to browse the library and extract the sticker IDs. You will use them later to generate your own.

### Crafting your own stickers

To create a sticker URL, you need to append the sticker ID to the avatar ID. Let's say that you want to render the sticker with ID `987cfa1b-cbd3-49c6-b4de-5d62bdd31bf8`. The format for custom stickers is:

```
https://sdk.bitmoji.com/me/sticker/<avatar_id>/<sticker_id>
```

So, for avatar ID `AWVscGhvWez65Dau7FgShF5VPT5C7A` and sticker ID `987cfa1b-cbd3-49c6-b4de-5d62bdd31bf8` the URL is:

```
https://sdk.bitmoji.com/me/sticker/AWVscGhvWez65Dau7FgShF5VPT5C7A/b9baff8d-2774-464d-b513-c1d6be51e56c
```

![](https://sdk.bitmoji.com/me/sticker/AWVscGhvWez65Dau7FgShF5VPT5C7A/b9baff8d-2774-464d-b513-c1d6be51e56c)

## Custom friendmoji stickers

### Building your friend network

To make friendmoji work, you'll need to take care of your app's friend graph on your own. Each Bitmoji user has an unique avatar ID specific to your application. You can get this identifier as detailed above in the `User Data` section.

### Crafting your own friendmoji stickers
If you have sticker IDs from our gallery with friendmoji support, you just need to add the friend id to the URL path to render the friendmoji:

```
https://sdk.bitmoji.com/me/sticker/<avatar_id>/<sticker_id>/<friend_id>
```

So, for avatar ID `AWVscGhvWez65Dau7FgShF5VPT5C7A`, sticker ID `af7653b1-080b-4bda-a3bd-1ea8e2f6a3cf` and friend ID `AWVscGhvI4X~eAfUWuGRCRWGNHerTQ` the URL is:

```
https://sdk.bitmoji.com/me/sticker/AWVscGhvWez65Dau7FgShF5VPT5C7A/0e7afb8b-14de-4d30-8498-a00cfe20fa4d/AWVscGhvI4X~eAfUWuGRCRWGNHerTQ
```

![](https://sdk.bitmoji.com/me/sticker/AWVscGhvWez65Dau7FgShF5VPT5C7A/0e7afb8b-14de-4d30-8498-a00cfe20fa4d/AWVscGhvI4X~eAfUWuGRCRWGNHerTQ)

### Single stickers featuring your friend

You can use the `friend_id` instead of the user's `avatar_id` to render a friend in the context of your app. A classic example of this use case is a conversation list in a chat app where you want to show to the user their friend's avatar. The URLs are similar to the user ones. The only diffence is they must include the `friend=1` query parameter.

So for sticker ID `987cfa1b-cbd3-49c6-b4de-5d62bdd31bf8` and friend ID `AWVscGhvI4X~eAfUWuGRCRWGNHerTQ` the URL is:
```
https://sdk.bitmoji.com/me/sticker/AWVscGhvI4X~eAfUWuGRCRWGNHerTQ/8e540795-8684-4cf1-853c-af2a41ec9abb?friend=1
```

![](https://sdk.bitmoji.com/me/sticker/AWVscGhvI4X~eAfUWuGRCRWGNHerTQ/8e540795-8684-4cf1-853c-af2a41ec9abb?friend=1)

## Parameters

Any of the previous URLs can use parameters to adapt the images to different scenarios

### Format

An extension can be added to the URLs to configure the image format. The available values are:
- `png`
- `webp` (default)

Example:

```
https://sdk.bitmoji.com/me/sticker/AUxuZkpWrWuY1hVhLWz6kFNO~bJ8mg/987cfa1b-cbd3-49c6-b4de-5d62bdd31bf8.png

https://sdk.bitmoji.com/me/sticker/AUxuZkpWrWuY1hVhLWz6kFNO~bJ8mg/987cfa1b-cbd3-49c6-b4de-5d62bdd31bf8.webp
```

### Size

A query parameter can be used to request different image sizes. It is recommended to use `thumbnail` when creating a picker interface for faster loading:
- `thumbnail`
- `default`
- `large`

```
https://sdk.bitmoji.com/me/sticker/AWVscGhvWez65Dau7FgShF5VPT5C7A/5ee3832d-7743-43c8-b6d7-ea47f11a1798?size=thumbnail

https://sdk.bitmoji.com/me/sticker/AWVscGhvWez65Dau7FgShF5VPT5C7A/5ee3832d-7743-43c8-b6d7-ea47f11a1798?size=default

https://sdk.bitmoji.com/me/sticker/AWVscGhvWez65Dau7FgShF5VPT5C7A/5ee3832d-7743-43c8-b6d7-ea47f11a1798?size=large
```

![](https://sdk.bitmoji.com/me/sticker/AWVscGhvWez65Dau7FgShF5VPT5C7A/5ee3832d-7743-43c8-b6d7-ea47f11a1798?size=thumbnail)

![](https://sdk.bitmoji.com/me/sticker/AWVscGhvWez65Dau7FgShF5VPT5C7A/5ee3832d-7743-43c8-b6d7-ea47f11a1798?size=default)

![](https://sdk.bitmoji.com/me/sticker/AWVscGhvWez65Dau7FgShF5VPT5C7A/5ee3832d-7743-43c8-b6d7-ea47f11a1798?size=large)
