
# slackin

A little server that enables public access
to a Slack server. Like Freenode, but on Slack.

It provides

- A landing page you can point users to fill in their
  emails and receive an invite (`http://slack.yourdomain.com`)
- An `<iframe>` badge to embed on any website
  that shows connected users in *realtime* with socket.io.
- A SVG badge that works well from static mediums
  (like GitHub README pages)

Read more about the [motivations and history](http://rauchg.com/slackin) behind Slackin.

## How to use

### Server


#### Heroku

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/rauchg/slackin/tree/0.5.1)

#### OpenShift

1. Create your [OpenShift](https://openshift.redhat.com) account and [install their client tools](https://developers.openshift.com/en/managing-client-tools.html)

1. Create a new OpenShift application for slackin:

  ``` shell
  rhc app create slackin https://raw.githubusercontent.com/kyrylkov/openshift-iojs/master/metadata/manifest.yml
  ```

1. Identify the following variable values:

  - `SLACK_SUBDOMAIN`: Your Slack's subdomain (the `this` part in `this.slack.com`),
  - `SLACK_API_TOKEN`: A Slack API token (find it on https://api.slack.com/web)
  - `SLACK_CHANNELS` (optional): Comma-separated list of single guest channels to invite them to (leave blank for a normal, all-channel invite). In order to make this work, you have to have a paid account. You'll only be able to invite as many people as your number of paying members times 5.
  
  And set them for the app using:

  ``` shell
  rhc set-env -a slackin SLACK_API_TOKEN='' SLACK_SUBDOMAIN='' SLACK_CHANNELS=''
  ```

1. If you'd like a custom domain, run:

  ``` shell
  rhc alias-add slack.YOUR_DOMAIN.COM -a slackin
  ```
  
  Then create CNAME record with your DNS host pointing `slack.YOUR_DOMAIN.COM` to `slackin-YOUR_OPENSHIFT_NAMESPACE.rhcloud.com`

1. Deploy slackin to your app:
  
  ``` shell
  git clone https://github.com/balupton/slackin.git
  cd slackin
  git remote add openshift `rhc app-show slackin | grep Git | sed 's/^.*ssh/ssh/'`
  git push openshift feature-openshift:slackin --force
  cd ..
  rm -Rf slackin
  ```

1. You should be all good now! Check the logs of your app with:
  
  ``` shell
  rhc tail -a slackin
  ```


#### Custom


Install it and launch it on your server:

```bash
$ npm install -g slackin
$ slackin "your-slack-subdomain" "your-slack-token"
```

You can find your API token at [api.slack.com/web](https://api.slack.com/web) – note that the user you use to generate the token must be an admin. You may want to create a dedicated `@slackin-inviter` user (or similar) for this.

The available options are:

```
Usage: slackin [options] <slack-subdomain> <api-token>

Options:

  -h, --help               output usage information
  -V, --version            output the version number
  -p, --port <port>        Port to listen on [$PORT or 3000]
  -c, --channels [<chan>]  One or more comma-separated channel names to allow single-channel guests [$SLACK_CHANNELS]
  -i, --interval <int>     How frequently (ms) to poll Slack [$SLACK_INTERVAL or 1000]
  -s, --silent             Do not print out warns or errors
```

**Important: if you use Slackin in single-channel mode, you'll only be
able to invite as many external accounts as paying members you have
times 5. If you are not getting invite emails, this might be the reason.
Workaround: sign up for a free org, and set up Slackin to point to it
(all channels will be visible).**

### Realtime Badge

[![](https://cldup.com/IaiPnDEAA6.gif)](http://slack.socket.io)

```html
<script async defer src="http://slack.yourdomain.com/slackin.js"></script>
```

or for the large version, append `?large`:

```html
<script async defer src="http://slack.yourdomain.com/slackin.js?large"></script>
```

### SVG

[![](https://cldup.com/jWUT4QFLnq.png)](http://slack.socket.io)

```html
<img src="http://slack.yourdomain.com/badge.svg">
```

### Landing page

[![](https://cldup.com/WIbawiqp0Q.png)](http://slack.socket.io)

Point to `http://slack.yourdomain.com`.

**Note:** the image for the logo of the landing page
is retrieved from the Slack API. If your organization
doesn't have one configured, it won't be shown.

## API

Requiring `slackin` as a module will return
a `Function` that creates a `HTTP.Server` instance
that you can manipulate.

```js
require('slackin')({
  token: 'yourtoken', // required
  interval: 1000,
  org: 'your-slack-subdomain', // required
  channels: 'channel,channel,...' // for single channel mode
  silent: false // suppresses warnings
}).listen(3000);
```

This will show response times from Slack and how many
online users you have on the console.

By default logging is enabled.

## Developing

Slackin's server side code is written in ES6. It uses babel to transpile the 
ES6 code to a format node understands. After cloning Slackin, you should 
install the prerequisite node libraries with npm:

```bash
$ npm install
```

After the libraries install, the postinstall script will run make to invoke
babel on the source. It is important to run make manually after updating any 
files in lib/ to update the versions in node/.

## Credits

- The SVG badge generation was taken from the
excellent [shields](https://github.com/badges/shields) project.
- The button CSS is based on
[github-buttons](https://github.com/mdo/github-buttons).

## License

MIT
