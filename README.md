<p align="center">
  <img src="https://cdn.qoilo.com/tK0y.gif">
</p>

<img src="https://img.shields.io/badge/Hosting-Hyper%20Expert-blue?style=flat-square&logo=server-fault&logo-color=white&link=https://qoilo.com/hosting" alt="Hosting"> <img src="https://img.shields.io/badge/Support-Discord-blue?style=flat-square&logo=discord&color=7289DA&logoColor=7289DA&link=https://discord.gg/h8qthfS" alt="Discord">  <img src="https://img.shields.io/badge/Version-4.5.2-red?style=flat-square&link=https://qoilo.com/hosting" alt="Version">

# ShareS - A Nodejs ShareX Upload Server
## Features

- ### Image/Video/General file uploading
- ### Text (With [Syntax Highlighting](https://highlightjs.org/)) [[Example](https://cdn.qoilo.com/s62pq)]
- ### URL shortening + a front end for the URL shortener as well
- ### [Markdown rendering](https://github.com/jonschlinkert/remarkable) files [[Example](https://cdn.qoilo.com/LxiR)]
- ### Logging via a Discord channel
- ### Password protected uploading (Uploads that require file-specific password to view/download)
- ### Server Administration using Discord bot commands
- ### Front end upload page [[Example](https://cdn.qoilo.com)]
- ### Password protected gallery page (password is admin key)
- ### Showcase images (image display pages that show metadata for photography) [[Example](https://cdn.qoilo.com/KH5z)]

## Installation (Ubuntu 16.04 Server)
```sh
https://github.com/TannerReynolds/ShareX-Upload-Server.git
cd ShareX-Upload-Server
chmod +x install.sh
./install.sh
```

## Configuration

In the files you downloaded from this repository, you will see a file called `config.json` 
You must fill this out for the webserver to work properly. Below explains the configuration file and what each part does

```js
{
  "key": [""], // Password(s) for private uploading
  "domain": "*.example.com", // Domain server will use. Will error if domain not used in request. Place "*" as the subdomain to enable wildcard subdomains for the webserver.
  "puploadKeyGenLength": 64, // Amount of characters server should use for pupload files
  "public": false, // Disables auth and does not render a password field for /upload
  "maxUploadSize": 50, // max upload size for non-admins using regular key in MB
  "markdown": true, // enables markdown rendering (upload whole .md file for render)
  "port": 80, // port to listen on
  "secure": true, // Whether or not you want https. (make sure key and cert.pem are in src directory)
  "securePort": 443, // Port to use when secure is true
  "ratelimit": 1000, // Ratelimit for POSTing in milliseconds
  "allowed":[
    "png", "jpg", "gif", "mp4", "mp3", "jpeg", "tiff", "bmp", "ico", "psd", "eps", "raw", "cr2", "nef", "sr2", "orf", "svg", "wav", "webm", "aac", "flac", "ogg", "wma", "m4a", "gifv"
  ], // Allowed file types for non-admins
  "admin":{
    "key": [""], // Admin password(s) for uploading & for gallery access
    "maxUploadSize": 1024, // Max upload size for admin in MB
    "allowed": [
    "png", "jpg", "gif", "mp4", "mp3","jpeg", "tiff", "bmp", "ico", "psd", "eps", "raw", "cr2", "nef", "sr2", "orf", "svg", "wav", "webm", "aac", "flac", "ogg", "wma", "m4a", "gifv", "html"
     ] // Allowed file types for admins
  },
  "paste": {
    "maxUploadSize": 20 // allowed paste upload size in MB
  },
  "discordToken": "", // Discord bot token
  "discordAdminIDs": ["discord IDs of people who can run commands go here", "Like this"], // User IDs in an array
  "discordChannelID": "", // Channel ID for monitoring uploads to
  "prefix": "" // Bot Prefix
}
```

## Running The Server
Once you've properly configured your server, you can run `node index.js` in the src folder to start the server.
You can keep your server running forever if you use a process manager, like pm2. pm2 installs along with your server if you used the install.sh script to install your server. Otherwise you can run `npm i -g pm2` to install pm2. Then you can run your server by running `pm2 start index.js`, and monitor logs and such using `pm2 monit`

### Note: Nginx/reverse proxy users
If you're configuring this webserver to run through an Nginx reverse proxy, make sure you add these lines to your reverse proxy config
```
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection 'upgrade';
proxy_set_header Host $host;
proxy_cache_bypass $http_upgrade;
```
This is generally some things you want to add to your config, and is what's actually required for ShareS to work properly. This is because ShareS returns uploads like `[http/https]://[requested url]/[filename]` and since you're running ShareS through a reverse proxy, unless you're passing along the *original* headers, ShareS is most likely just going to send you something like `http://[server's real ip address]/[filename]`

### Note: Users of multiple domains
If you have multiple domains pointed to this webserver, only one (can include wildcard subdomain) can be used, **unless** the domain setting is set to just a single * like so: `"domain": "*",`. This means that any domain will be accepted as a valid domain by the server, regardless of subdomain.

## Setting up Discord logging
if you wish to log your webserver's activity in a Discord channel for whatever reason, you can.
[Here is information on how to setup a bot account and get the information needed for Discord logging](https://github.com/reactiflux/discord-irc/wiki/Creating-a-discord-bot-&-getting-a-token)

## Configuring Your ShareX Client
 - [Download this repository to the PC your ShareX is on](https://github.com/TannerReynolds/ShareX-Upload-Server/archive/master.zip)
 - Navigate to `Destinations -> Custom Uploader Settings`
 - Import the sxcu files from the downloaded zip one by one
 - Properly Configure Template
 - Change `Destination Location`

### Configuring for Password Protected Uploading
- Add a field to your body called `pupload`, and then make the value whatever you want the password to be
- Upload something, and the upload will give you a url to the authentication page
- Type in your password, and it will display/download the file!

#### Auto Password Generation
In addition to being able to use any password you want for puploads, if you type in `*random*` as your pupload field, the server will automatically generate a password for you. This password will include letters, numbers, and special characters. It will generate a key based on the length you specify in your config (puploadKeyGenLength). When making requests, the server will return the image URL with the key like so `URL: https://qoilo.com/lhHr | KEY: Np$[CBk>X[c^YY{MDlCHH0|Qfm1uK0*lld^Mi$f4d62R5x6C2>~yaL}3*QYnziuZ`

### Showcase Field
 - Add a field to your body called `showCase`, and then make the value `true`
 - Upload an image
 - Click the image to view image's metadata like camera, lens, iso, shutter speed, etc.
 - Requires extra software to be installed to your server, called [Exiftool](https://www.sno.phy.queensu.ca/~phil/exiftool/index.html) to read metadata from uploaded images. The install file will automatically install this software on ubuntu.
 - Windows servers using this feature wil need the Windows executable for exiftool and it will need to be added to your environment variables or ShareS will throw errors on upload and return 404s
 
## Contributing
### Pull Requests
 - Be sure you properly lint your files prior to making a pull request. eslint file available
 - Do your own testing
 - Properly comment your code. My code isn't really commented on because it's my code and I understand how everything fits together and works. It isn't a very large project. You will need to comment on your code though so that I could understand going forward when more features get added
 - Not all features are going to be added. Just because you code something that may be useful to you, that does not mean it will for sure be added. 
 - **Update the version of ShareS when you create a pull request. If the current version at the time of your PR is** `4.5.6`**, make sure to change the shield in the README and the package.json file to say version** `4.5.7`**. If the version is** `4.5.10`**, then update it to** `4.5.11`**. Only update the last digit. Updates/PRs that are not code (README.md, install.sh, sxcu files, package.json, etc.) should not constitute an updated version number for the project, so PRs for those specific files will not increment the version number**

### Bug Reports
 - Follow the issue template format or your issue will likely not be solved. If you do not follow the format I won't have enough information to diagnose the problem and fix it. 
  - If I ask questions and you do not respond on the thread, and I'm not experiencing the issue / am not able to replicate the issue, I will close the issue thread, and you'll have to make another one if the problem persists. 
  - Please make sure your system setup isn't the cause of your issue. This webserver can be ran through several different pieces of software, reverse proxies, operating systems, network types, just please make sure that you havent made in error in setting up your system before submitting the issue. We do not help people with every combination of setup that exists, if you're using some weird OS, runnning ShareS through your own custom reverse proxy, don't expect much support from us if it doesn't work.
  - Ensure your config isn't the issue. Most issues people have with setting up the server and experiencing crashes are due to an error in how you made your config file. Be sure to read the descriptions for each one and use the correct data type (array, string, number, array with strings, etc.)

## Credits
#### [Ken](https://github.com/NotWeeb) - Initial File Uploader
#### [Aetheryx](https://github.com/aetheryx) - Webserver Structure
#### [Jaex](https://github.com/Jaex) - ShareX
#### [FancyApps](https://github.com/fancyapps/fancybox) - Gallery lightbox script for displaying images, videos and more


#### <a href="https://qoilo.com/hosting">Cheap Hosting Options For Your Uploader</a>
