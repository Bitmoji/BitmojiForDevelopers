# Bitmoji Search API: Web Sample (plain)

## Generate the project

Create a directory for your project and minimal `index.html` file

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bitmoji Picker</title>
</head>
<body>
    <h1>Hello Bitmoji</h1>
</body>
</html>
```

You need to serve the website through a server. An easy option is using Python

```bash
$ python3 -m http.server
```

you should see a message like:

```
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/)
```

Open `http://localhost:8000` in your browser to verify you see your initial web.

## Configuring OAuth

We'll be using Login Kit. For more information check the [website](https://kit.snapchat.com/login-kit).

### Configure the app in the Developer Portal
Create your Snap Kit app in `https://kit.snapchat.com/portal/`. Make sure to add Bitmoji Avatar permission.

In the Authorization section, enable `Web` for the Developement Environment. Next add `http://localhost:8000` to the Redirect URLs section.

> Note: If you want to test with users other than your own, make sure to add them at the bottom in the test users section.

Finally, copy the `CLIENT ID` for the development enviroment. You'll need it in the next step.

### Configure the JS Login Kit library

Going back to `index.html`, replace `<h1>Hello Bitmoji</h1>` with the Login Kit boilerplate

```html
<div id="display_name"></div>
<img id="bitmoji" />
<div id="external_id"></div>
<hr />
<div id="my-login-button-target"></div>
<script>
    window.snapKitInit = function () {
        var loginButtonIconId = 'my-login-button-target';
        // Mount Login Button
        snap.loginkit.mountButton(loginButtonIconId, {
            clientId: 'YOUR_CLIENT_ID',
            redirectURI: 'YOUR_REDIRECT_URI',
            scopeList: [
                'user.display_name',
                'user.bitmoji.avatar',
                'user.external_id'
            ],
            handleResponseCallback: function () {
                snap.loginkit.fetchUserInfo().then(
                    function (result) {
                        console.log("User info:", result.data.me);
                        document.getElementById("display_name").innerHTML =
                        result.data.me.displayName;
                        document.getElementById("bitmoji").src =
                        result.data.me.bitmoji.avatar;
                        document.getElementById("external_id").src =
                        result.data.me.externalId;
                    },
                    function (err) {
                        console.log(err); // Error
                    }
                );
            },
        });
    };
    // Load the SDK asynchronously
    (function (d, s, id) {
        var js,
            sjs = d.getElementsByTagName(s)[0];
        if (d.getElementById(id)) return;
        js = d.createElement(s);
        js.id = id;
        js.src = "https://sdk.snapkit.com/js/v1/login.js";
        sjs.parentNode.insertBefore(js, sjs);
    })(document, "script", "loginkit-sdk");
</script>
```

Now, replace in that boilerplate `YOUR_CLIENT_ID` with the value that you copied from the Developer Portal in the previous step. Also replace `YOUR_REDIRECT_URI` with `http://localhost:8000`.

If you reload the page you should see the `Continue with Snapchat` button. You can test that going through the OAuth process, after clicking it, shows your name and avatar on the webpage.

---

# Fetching content from Bitmoji Direct

## Extracting the token from Login Kit

To hit the Bitmoji Direct endpoints, you'll need a valid Login Kit token. The boilerplate above uses it behind the scenes. Let's change the callback handler to fetch the token instead

```js
handleResponseCallback: function () {
    snap.loginkit.fetchUserInfo().then(
        function (result) {
            const token = snap.loginkit.getSharedDataAccessToken();
            loadBitmojiContent(token);
        },
        function (err) {
            console.log(err); // Error
        }
    );
}
```

Replace the Login Kit the boilerplate markup to test the data
```html
<div id="display_name"></div>
<img id="bitmoji" />
<div id="external_id"></div>
<hr />
```

with
```html
<div id="sticker-picker">
     <div id="stickers"></div>
</div>
```

## Showing popular stickers

### Fetching the content
In the new callback a non-existent `loadBitmojiContent(token)` function is being called. Let's create a `script` tag with its code.

Bitmoji Direct provides a specific pack with the `https://bitmoji.api.snapchat.com/direct/pack/<PACK_ID>` endpoint. The popular stickers are in a pack called `popular` so the endpoint is `https://bitmoji.api.snapchat.com/direct/pack/popular`. To authenticate the user, you need to pass the token in the `Authorization` header.

```html
<script>
        const HOST = 'https://bitmoji.api.snapchat.com';

        async function loadBitmojiContent(token) {
            console.log(`Token ${token}`);
            const response = await fetch(`${HOST}/direct/pack/popular`, {
                headers: {
                    Authorization: `Bearer ${token}`
                }
            });
            const json = await response.json();
            const stickers = json.data;
            console.log(stickers);
        }
</script>
```

The `data` field contains the payload. Each sticker has a `uri` property with an image URL. That's all it is needed to render the content.

### Displaying the content

With the information from the previous step you can easily display all the stickers with adding this code to the function

```js
const stickerPicker = document.getElementById('stickers');
stickers.forEach(sticker => {
    const image = new Image(100, 100);
    image.src = sticker.uri;
    stickerPicker.appendChild(image);
});
```

> Note: This is an extremely inneficient way of displaying the data. We are focused on the minimal code that you can then adapt to your own libraries and frameworks.

With this code you should now see a lot of stickers featuring your avatar

## Providing Search functionality

### Adding controls

With a text input and a button is easy to implement a basic search UI. Update the sticker-picker div to

```html
<div id="sticker-picker">
    <form id="search-box" style="display:none; margin: 10px">
        <input id="searchbox"/>
        <input type="submit" id="search-button" value="Search"/>
    </form>
    <div id="stickers"></div>
</div>
```

Bitmoji Direct provides search functionality through the `https://bitmoji.api.snapchat.com/direct/search?q=YOUR_SEARCH_TERM` endpoint. To authenticate the user, you need to pass the token in the `Authorization` header.

You can add the following code after the sticker loading code to provide basic search functionality

```js
const searchBox = document.getElementById('search-box');
searchBox.style.display = 'block';
const searchButton = document.getElementById('search-button');

searchButton.addEventListener('click', async (e) => {
    e.preventDefault();
    const searchText = document.getElementById('searchbox').value;
    const response = await fetch(`${HOST}/direct/search?q=${searchText}`, {
        headers: {
            Authorization: `Bearer ${token}`
        }
    });
    const json = await response.json();
    const stickers = json.data;

    stickerPicker.childNodes.forEach(node => node.remove())
    stickers.forEach(sticker => {
        const image = new Image(100, 100);
        image.src = sticker.uri;
        stickerPicker.appendChild(image);
    });
});
```

If you type `hello` in the search box and hit enter the content should be replaced with stickers matching your search term.

### Clean up the code

After our last step there is a significant amount of duplication. You can simplify significantly refactoring to

```js
const HOST = 'https://bitmoji.api.snapchat.com';

async function fetchFromAPI(url, token) {
    const response = await fetch(url, {
        headers: {
            Authorization: `Bearer ${token}`
        }
    });
    const json = await response.json();
    return json.data;
}

function getPopularStickers(token) {
    return fetchFromAPI(`${HOST}/direct/pack/popular`, token);
}

function getStickerSearchResults(token, searchText) {
    return fetchFromAPI(`${HOST}/direct/search?q=${searchText}`, token);
}

function renderStickers(stickers) {
    const stickerPicker = document.getElementById('stickers');
    stickerPicker.innerHTML = '';
    stickers.forEach(sticker => {
        const image = new Image(100, 100);
        image.src = sticker.uri;
        stickerPicker.appendChild(image);
    });
}

async function loadBitmojiContent(token) {
    const stickers = await getPopularStickers(token);
    renderStickers(stickers);

    const searchBox = document.getElementById('search-box');
    searchBox.style.display = 'block';

    const searchButton = document.getElementById('search-button');
    searchButton.addEventListener('click', async (e) => {
        e.preventDefault();
        const searchText = document.getElementById('searchbox').value;
        const searchResultStickers = await getStickerSearchResults(token, searchText);
        renderStickers(searchResultStickers);
    });
}
```

## Providing pack browsing functionality

### Showing available packs

Bitmoji Direct also groups content in a set of packs. In a similar fashion to `popular`, you can get stickers grouped by a given tag such as `yes`, `hello` or `happy`. To get a set of tags you can suggest to your users you can call the `https://bitmoji.api.snapchat.com/direct/packs` endpoint.

Update the sticker `div` to

```html
<div id="sticker-picker">
    <form id="search-box" style="display:none; margin: 10px">
        <input id="searchbox"/>
        <input type="submit" id="search-button" value="Search"/>
    </form>
    <div id="tags"></div>
    <div id="stickers"></div>
</div>
```

and now load the packs into `<div id="tags">` with

```js

(...)

function getPacks(token) {
    return fetchFromAPI(`${HOST}/direct/packs`, token);
}

(... in loadBitmojiContent)
const packs = await getPacks(token);
const tagsContainer = document.getElementById('tags');
packs.forEach(pack => {
    const packLink = document.createElement('a');
    packLink.innerHTML = pack.name;
    packLink.style.marginRight = '24px';
    packLink.href = "#";
    tagsContainer.appendChild(packLink);
});
```

### Showing stickers within pack when clicking

Now you can add the event handler to show the relevant content when clicking one of the tags. To do so, the `id` field within the pack is used. `popular` is just another pack `id` field. Let's refactor `getPopularSticker` to accept any packId

```js
function getPackStickers(token, packId) {
    return fetchFromAPI(`${HOST}/direct/pack/${packId}`, token);
}

(... in loadBitmojiContent)
const stickers = await getPackStickers(token, 'popular');
```

Finally, let's add the click handler for the tags

```js
packLink.addEventListener('click', async (e) => {
    e.preventDefault();
    const stickers = await getPackStickers(token, pack.id);
    renderStickers(stickers);
});
```

## Picking a sticker

When users select a sticker an action like should happen. For instance, you may want to share the sticker in a 1:1 conversation in a chat application. To do this the same URI used to display the sticker preview may be used.

Let's update the app UI to include an area to display the selected image
```html
<div id="app-container" style="display:grid; grid-template-columns: 50vw 50vw">
    <div id="sticker-picker">
        <form id="search-box" style="display:none; margin: 10px">
            <input id="searchbox" />
            <input type="submit" id="search-button" value="Search" />
        </form>
        <div id="tags"></div>
        <div id="stickers"></div>
    </div>
    <div>
        <img id="share-area" style="width: 100%"/>
    </div>
</div>
<div id="my-login-button-target"></div>
```

Now in the function where we render the stickers, we can add a handler to call a function with the click event is triggered:
```js
function renderStickers(stickers) {
    const stickerPicker = document.getElementById('stickers');
    stickerPicker.innerHTML = '';
    stickers.forEach(sticker => {
        const image = new Image(100, 100);
        image.src = sticker.uri;
        image.onclick = () => shareSticker(sticker);
        stickerPicker.appendChild(image);
    });
}

function shareSticker(sticker) {
    document.getElementById('share-area').src = sticker.uri;
}
```

If you test this code it will work, but the shared image will apear pixelated. To allow faster loading times in your picker, stickers are loaded on their `thumbnail` size. This may be undesirable for the sharing use case where a higher image resolution may be required. It is trivial to do this. Stickers have a number of query parameters attached to them, you can manipulate the `size` parameter to load a bigger size image. The available options are:
- `default`: 400x400 pixels
- `large`: 800x800 pixels

Adding the size parameter to the example:
```js
function shareSticker(sticker) {
    document.getElementById('share-area').src = sticker.uri.replace('size=thumbnail', 'size=default');
}
```

Bitmoji may be notified that a sticker has been picked to share. A stub example could be:

```js
function shareSticker(sticker) {
    document.getElementById('share-area').src = sticker.uri.replace('size=thumbnail', 'size=default');
    fetch(`${HOST}/direct/event/${sticker.on_share}`, { method: 'post' });
}
```

## Customizing content

### Locale
The `/packs`, `/pack/{packId}` and `/search` endpoints support a `locale` query parameter. The content will be returned in the locale of your choice if supported. I will default to English.

The locales supported currently are: `en`, `ar`, `de`, `es`, `fr`, `it`, `ja`, `ko` and `pt`.

If you don't provide an explicit `locale` parameter but the `accept-language` HTTP header is set, the API will resolve based on the header.

### Time
The `/packs` and `/pack/{packId}` endpoints support a `time` parameter.

Bitmoji content is variable depending on time of day. To be able to display the right content to your users you can send a date string in the `time` query parameter and timely content will be returned if available.

Example of valid time strings are `2020-10-15T13:05:05`(no timezone) or `2020-10-15T18:02:04-04:00`(with timezone).

### Pagination

The `/pack/{packId}` and `/search` endpoints support optional `limit` and `page` query parameters to paginate through the returned data. `limit`
should be a positive integer while `page` should be 0 or positive. If only a `limit` param is passed, the `page` defaults to 0. Specifying only
`page` doesn't affect the results.

To help with pagination, the result from Bitmoji direct API also returns metadata in the JSON result, this can be ignored if you aren't using
pagination. This includes the field `next` - URL path with query pointing to the next page, `prev` URL path with query pointing to the previous
page and `total` - the total number of elements for this query (this can be used to set the page size). `next` and `prev` are null if
pagination params weren't specified or if no `next` or `prev` page exists. Note that `next` and `prev` preserve original query params.

As an example, the result for `https://bitmoji.api.snapchat.com/direct/search?q=hi&locale=en&limit=10&page=2` is as follows:

```json
{
    "data": [...],
    "metadata": {
        "next": "/direct/search?q=hi&locale=en&limit=10&page=3",
        "prev": "/direct/search?q=hi&locale=en&limit=10&page=1",
        "total": 56
    }
}
```

## Full code for reference

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bitmoji Picker</title>
</head>

<body>
    <div id="app-container" style="display:grid; grid-template-columns: 50vw 50vw">
        <div id="sticker-picker">
            <form id="search-box" style="display:none; margin: 10px">
                <input id="searchbox" />
                <input type="submit" id="search-button" value="Search" />
            </form>
            <div id="tags"></div>
            <div id="stickers"></div>
        </div>
        <div>
            <img id="share-area" style="width: 100%"/>
        </div>
    </div>
    <div id="my-login-button-target"></div>
    <script>
        window.snapKitInit = function () {
            var loginButtonIconId = 'my-login-button-target';
            // Mount Login Button
            snap.loginkit.mountButton(loginButtonIconId, {
                clientId: 'YOUR_CLIENT_ID',
                redirectURI: 'YOUR_REDIRECT_URI',
                scopeList: [
                    'user.display_name',
                    'user.bitmoji.avatar',
                    'user.external_id'
                ],
                handleResponseCallback: function () {
                    snap.loginkit.fetchUserInfo().then(
                        function (result) {
                            const token = snap.loginkit.getSharedDataAccessToken();
                            loadBitmojiContent(token);
                        },
                        function (err) {
                            console.log(err); // Error
                        }
                    );
                },
            });
        };
        // Load the SDK asynchronously
        (function (d, s, id) {
            var js,
                sjs = d.getElementsByTagName(s)[0];
            if (d.getElementById(id)) return;
            js = d.createElement(s);
            js.id = id;
            js.src = "https://sdk.snapkit.com/js/v1/login.js";
            sjs.parentNode.insertBefore(js, sjs);
        })(document, "script", "loginkit-sdk");
    </script>

    <script>
        const HOST = 'https://bitmoji.api.snapchat.com';

        async function fetchFromAPI(url, token) {
            const response = await fetch(url, {
                headers: {
                    Authorization: `Bearer ${token}`
                }
            });
            const json = await response.json();
            return json.data;
        }

        function getPackStickers(token, packId) {
            return fetchFromAPI(`${HOST}/direct/pack/${packId}`, token);
        }

        function getStickerSearchResults(token, searchText) {
            return fetchFromAPI(`${HOST}/direct/search?q=${searchText}`, token);
        }

        function getPacks(token) {
            return fetchFromAPI(`${HOST}/direct/packs`, token);
        }

        function renderStickers(stickers) {
            const stickerPicker = document.getElementById('stickers');
            stickerPicker.innerHTML = '';
            stickers.forEach(sticker => {
                const image = new Image(100, 100);
                image.src = sticker.uri;
                image.onclick = () => shareSticker(sticker);
                stickerPicker.appendChild(image);
            });
        }

        function shareSticker(sticker) {
            document.getElementById('share-area').src = sticker.uri.replace('size=thumbnail', 'size=default');
            fetch(`${HOST}/direct/event/${sticker.on_share}`, { method: 'post' });
        }

        async function loadBitmojiContent(token) {
            const stickers = await getPackStickers(token, 'popular');
            renderStickers(stickers);

            const searchBox = document.getElementById('search-box');
            searchBox.style.display = 'block';

            const searchButton = document.getElementById('search-button');
            searchButton.addEventListener('click', async (e) => {
                e.preventDefault();
                const searchText = document.getElementById('searchbox').value;
                const searchResultStickers = await getStickerSearchResults(token, searchText);
                renderStickers(searchResultStickers);
            });

            const packs = await getPacks(token);
            const tagsContainer = document.getElementById('tags');
            packs.forEach(pack => {
                const packLink = document.createElement('a');
                packLink.innerHTML = pack.name;
                packLink.style.marginRight = '24px';
                packLink.href = "#";
                packLink.addEventListener('click', async (e) => {
                    e.preventDefault();
                    const stickers = await getPackStickers(token, pack.id);
                    renderStickers(stickers);
                });
                tagsContainer.appendChild(packLink);
            });
        }
    </script>
</body>

</html>

```
