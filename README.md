# Slash Webtasks: Extend Slack With Node.js

[Install on your Slack team](https://webtask.io/slack)  
[File an issue](https://github.com/auth0/slash/issues)  

## Hello, world

After you have [installed Slash Webtasks](https://webtask.io/slack) on your Slack team, you can create your first webtask by typing 

```
/wt make hello
``` 

in any Slack channel, and following the provided link to edit the code in Webtask Editor. The simplest webtask will immediately return a response: 

```javascript
module.exports = (ctx, cb) => cb(null, { text: 'Hello, world!' });
```

You can invoke your webtask from any Slack channel with `/wt hello`. 

Type in `/wt help` to see all available operations. 

## Inputs and outputs

The `ctx.body` input parameter contains the payload which Slack sends to your webtask, e.g.:

```
{
 "team_id": "T025590N6",
 "team_domain": "auth0",
 "channel_id": "D1KFTMMTJ",
 "channel_name": "directmessage",
 "user_id": "U02FMKT1L",
 "user_name": "tomek",
 "command": "/wt run hello",
 "text": "foo bar baz",
 "response_url": "https://hooks.slack.com/commands/T025540N6/86862216608/4DNA0LVn6QG7xqfBhGSTIqoc"
}
```

Note that `ctx.body.text` contains the parameters to the webtask you typed in Slack following `/wt {webtask_name}`. This allows users to pass arbitrary input parameters. Check out [Slack documentation](https://api.slack.com/slash-commands#triggering_a_command) for more information about the input payload. 

The response your command sends back to Slack is a JSON object and is fully documented in [Slack docs](https://api.slack.com/slash-commands#responding_to_a_command). The two most commonly used properties are: 

```
{
  "text": "This is the text of the response",
  "response_type": "in_channel" // you can omit this if you want the response 
                                // to be only visible to the caller
}
```

## Sync and async responses

Slack allows you only 3 seconds to send a synchronous response, otherwise it will report timeout of the slash command. If your processing requires more time, as is often the case when making API calls to external services, it is a good practice to send a brief synchronous response as soon as you enter your webtask, and then send an asynchronous response once the processing has finished. You have a lot of time to send a follow up asynchronous response as long as you responded synchronously before.

This sample webtask demostrates this pattern:

```javascript
module.exports = function (ctx, cb) {
  // You only have 3 seconds to respond synchronously to Slack before your request times out
  sync_response(cb, `:hourglass: Working on it...`);
  setTimeout(() => {
    // If your work is taking longer, you can respond asynchronously any number of times later
    async_response(ctx, `Hello, @${ctx.body.user_name}!`);
  }, 1000);
};

function async_response(ctx, text, only_caller) {
  require('superagent').post(ctx.body.response_url)
    .send({ text: text, response_type: only_caller ? undefined : 'in_channel' })
    .end();
}

function sync_response(cb, text, only_caller) {
  cb(null, { text: text, response_type: only_caller ? undefined : 'in_channel' });
}
```
