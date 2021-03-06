---
title: Elixir with Absinthe
order: 2
---

**Supported Elixir servers:** [Absinthe](https://github.com/absinthe-graphql/absinthe)

To get started with Engine, you will need to:
1. Instrument your server with an Elixir tracing agent that uses the Apollo Tracing format.
2. Configure and deploy the Engine proxy docker container.
3. Send requests to your service – you're all set up!

At this time, the only available option for running the Engine proxy with an Elixir server is to run the proxy in a standalone docker container.

_Interested in writing a sidecar Elixir package? [Get in touch](mailto:support@apollodata.com) with us!_

<h2 id="enable-apollo-tracing" title="Enable Apollo Tracing">1. Enable Apollo Tracing</h2>
You will need to instrument your Elixir server with a tracing package that follows the [Apollo Tracing](https://github.com/apollographql/apollo-tracing) format. Engine relies on receiving data in this format to create its performance telemetry reports.

This is our recommended Elixir package: https://github.com/sikanhe/apollo-tracing-elixir

<h2 id="configure-the-proxy" title="Configure the Proxy">2. Configure the Proxy</h2>
<h3 id="get-api-key" title="Get your API Key">2.1 Get your API Key</h3>
First, get your `Engine_API_Key` by creating a service on http://engine.apollographql.com/. You will need to log in and click "Add Service" to recieve your API key.

<h3 id="create-config-json" title="Create your Config.json">2.2 Create your Proxy's Config.json</h3>
The proxy uses a JSON object to get configuration information. If the configuration is passed the path to your file, that file will be watched for changes. Changes will cause the proxy to adopt the new configuration without downtime.

**Create a JSON configuration file:**

```
{
  "apiKey": "<ENGINE_API_KEY>",
  "logging": {
    "level": "INFO"
  },
  "origins": [
    {
      "http": {
        "url": "http://localhost:3000/graphql"
      }
    }
  ],
  "frontends": [
    {
      "host": "0.0.0.0",
      "port": 3001,
      "endpoint": "/graphql"
    }
  ]
}
```

**Configuration options:**
1. `apiKey`: The API key for the Engine service you want to report data to.
2. `logging.level` : Logging level for the proxy. Supported values are `DEBUG`, `INFO`, `WARN`, `ERROR`.
3. `origin.http.url` : The URL for your GraphQL server.
4. `frontend.host` : The hostname the proxy should be available on.
5. `frontend.port` : The port the proxy should bind to.
6. `frontend.endpoint` : The path for the proxy's GraphQL server . This is usually `/graphql`.

For full configuration details see [Proxy config](proto-doc.html).

<h3 id="run-the-proxy" title="Run the Proxy">2.3 Run the Proxy (Docker Container)</h3>
The Engine proxy is a docker image that you will deploy and manage separate from your server.

If you have a working [docker installation](https://docs.docker.com/engine/installation/), type the following lines in your shell (variables replaced with the correct values for your environment) to run the Engine proxy:
```
engine_config_path=/path/to/engine.json
proxy_frontend_port=3001
docker run --env "ENGINE_CONFIG=$(cat "${engine_config_path}")" \
  -p "${proxy_frontend_port}:${proxy_frontend_port}" \
  gcr.io/mdg-public/engine:2017.11-137-g908dbec6f
```

It does not matter where you choose to deploy and manage your Engine proxy. We run our own on Amazon's [EC2 Container Service](https://aws.amazon.com/ecs/).

We recognize that almost every team using Engine has a slightly different deployment environment, and encourage you to [contact us](mailto: support@apollodata.com) with feedback or for help if you encounter problems running the Engine proxy.

<h2 id="view-metrics-in-engine" title="View Metrics in Engine">3. View Metrics in Engine</h2>
Once your server is set up, navigate your new Engine service on https://engine.apollographql.com. Start sending requests to your Elixir server to start seeing performance metrics!
