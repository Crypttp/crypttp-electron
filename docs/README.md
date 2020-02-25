# Crypttp for Electron.js Wallets

## Installation
### 1.
```
npm i -s crypttp-parser
```

### 2.
Signup at [Dashboard](https://crypttp.com/dashboard)

Navigate to Settings/Wallet App

Set:

* Name

* Deeplink

* Discription

* Icon

* Available currencies

* Urls to AppStore

In addition this configuration will help us promote your wallet app. 

Every user that has no wallet installed while paying at Crypttp merchants will be redirected to a special page where user can find featured wallets

### 3.
Get your merchant id in dashboard

<img src="https://i.imgur.com/fyW4P6s.png" height="100%" />

Then let Crypttp know that wallet was installed on `firstInstallation`

You can use something like [Electron-first-run](https://github.com/joecohens/electron-first-run)
```JavaScript
// main process
const firstRun = require('electron-first-run');

const isFirstRun = firstRun()
if (isFirstRun) {
    const url = "https://api.crypttp.com/track/installation?id=<you merchant id from dashboard>"
    require("electron").shell.openExternal(url);
}
```

Or without 3rd party module
```JavaScript
var cmd = process.argv[1];

if (cmd == '--squirrel-firstrun') {
    const url = "https://api.crypttp.com/track/installation?id=<you merchant id from dashboard>"
    require("electron").shell.openExternal(url);
}
```

The code above will open default browser and make quick configuration that will take less than a second, then user will be routed back to app.

## Connecting to Crypttp merchants and payment proccessors

### 1. Edit `package.json`
```JSON
{
  "name": "your app name",
  "version": "1.0.0",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
  },
  ...
  // Example of your build 
  "build": {
    "appId": "your.app.id",
    "productName": "Your app name",
    "mac": {},
    "dmg": {},
    "win": {},
    "linux": {},
    "protocols": [
      {
        "name": "yourappname",
        "role": "Viewer",
        "schemes": [
          "yourappname",
          "crypttp"
        ]
      }
    ],
  }
}
```

### 2. Edit your `main.js`

```JavaScript
const crypttpParser = require('crypttp-parser')

// in the main browser window render add protocol handlers
mainWindow = new BrowserWindow({
    show: false,
    width: 1024,
    height: 728,
    minWidth: 992,
    minHeight: 600
});

function foo (params) {
    // foo - your function to load sending crypto page
}

let parsedCrypttpParams;
/*
{
    "id":"merchantId",
    "successUrl": "successUrl",
    "errorUrl": "errorUrl",
    "params":[
        [
            "Currency1",
            "Amount of Currency1 to send",
            "Address where to send",
            "custom data to include in transaction",
            "memo if required",
        ],
        [
            "params for Currency2"
        ],
        [
            "params for Currency3"
        ]
    ]
}
*/

// Protocol handler for win32 and linux
if (process.platform === 'win32' || process.platform === 'linux') {
    let deepurl = process.argv.slice(1);
    parsedCrypttpParams = crypttpParser(deepurl)
    foo(parsedCrypttpParams);
}

app.setAsDefaultProtocolClient('yourappname');
app.setAsDefaultProtocolClient('crypttp');

app.on('open-url', (event, url) => {
    event.preventDefault();
    parsedCrypttpParams = crypttpParser(url))
    foo(parsedCrypttpParams;
});

```

### 3. Update successfull transaction handler

This method help us track paid transactions and provide statistics to our merchants. 

Which in turn helps attract new merchants.

```JavaScript
Axios.post("https://api.crypttp.com/aqr/saveConfirmedHash", {
    id: parsedCrypttpParams.id,
    hash: hashOfConfirmedTransaction
})
```

### 4. Return user to the parent page

After step `3` add code to return user to the webpage or app from where user was routed to wallet.

That is important to provide good UX and to let merchant know that transaction was successfully made or failed.

**In the successfull transaction handler**

```JavaScript
const { shell } = require('electron')

shell.openExternal(parsedCrypttpParams.successUrl)
```

**In the failed transaction handler**

```JavaScript
const { shell } = require('electron')

shell.openExternal(parsedCrypttpParams.errorUrl)
```