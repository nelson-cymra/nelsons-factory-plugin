# Nelson's Factory for Claude Code

Send a feature or a fix to [Nelson's Factory](https://nodejs-1683750-6753929.cloudwaysnodeapps.com),
which builds, checks, reviews and merges it in your GitHub repo.

## Install

In Claude Code:

```
/plugin marketplace add nelson-cymra/nelsons-factory-plugin
/plugin install nelsons-factory@nelsons-factory
```

Then run `/mcp`, choose `plugin:nelsons-factory:factory`, and sign in with GitHub in the browser.

You need a repo with the factory's GitHub App installed and set up; the dashboard walks an owner
through that.

## Use

Say what you want ("get the factory to add a CSV export to the orders page"), or run
`/nelsons-factory:build add a CSV export to the orders page`. Claude reads the code, asks a question
or two, shows the brief, and sends it when you say **send**. Follow the job on the dashboard.

When the idea changes a screen, Claude also draws a mockup in the app's look, with sample data. In
the Claude Code desktop app it opens in the Browser pane; elsewhere you get the file to open. It
changes as you answer, goes with the brief, and the factory builds the screen to it.

## License

MIT.
