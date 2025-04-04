---
title: Expose and graph Kong AI Metrics
minimum_version: 3.8.x
badge: enterprise
---


This guide walks you through collecting AI metrics and sending them to Prometheus, and
setting up the Grafana dashboard.

## Overview

Kong's AI Gateway (AI Proxy) calls LLM-based services according to the settings of the AI Proxy plugin.
You can aggregate the LLM provider responses to count the number of tokens used by the AI Proxy.
If you have defined input and output costs in the models, you can also calculate cost aggregation.
The metrics details also expose if the requests have been cached by {{site.base_gateway}}, saving the cost of contacting the LLM providers, which improves performance.

Kong AI Gateway exposes metrics related to Kong and proxied upstream services in 
[Prometheus](https://prometheus.io/docs/introduction/overview/) 
exposition format, which can be scraped by a Prometheus Server.

Metrics tracked by this plugin are available on both the Admin API and Status
API at the `http://localhost:<port>/metrics`
endpoint. Note that the URL to those APIs is specific to your
installation. See [Accessing the metrics](#accessing-the-metrics) for more information.

The Kong [Prometheus plugin](/hub/kong-inc/prometheus/) records and exposes metrics at the node level. Your Prometheus
server will need to discover all Kong nodes via a service discovery mechanism,
and consume data from each node's configured `/metrics` endpoint.

## Grafana dashboard

AI Metrics exported by the plugin can be graphed in Grafana using a [drop-in
dashboard](https://grafana.com/grafana/dashboards/21162-kong-cx-ai/):

![AI Grafana Dashboard](/assets/images/products/gateway/vitals/grafana-ai-dashboard.png)

## Available metrics

{% if_version lte: 3.7.x %}
- **AI Requests**: AI requests sent to LLM providers.
  These are available per provider, model, cache, database name (if cached), and workspace.
- **AI Cost**: AI costs charged by LLM providers.
  These are available per provider, model, cache, database name (if cached), and workspace.
- **AI Tokens**: AI tokens counted by LLM providers.
  These are available per provider, model, cache, database name (if cached), token type, and workspace.
{% endif_version %}

{% if_version gte: 3.8.x %}
All the following AI LLM metrics are available per provider, model, cache, database name (if cached), embeddings provider (if cached), embeddings model (if cached), and workspace.

When `ai_llm_metrics` is set to true:
- **AI Requests**: AI request sent to LLM providers.
- **AI Cost**: AI Cost charged by LLM providers.
- **AI Tokens**: AI Tokens counted by LLM providers.
  These are also available per token type in addition to the options listed previously.
- **AI LLM Latency**: Time taken to return a response by LLM providers.
- **AI Cache Fetch Latency**: Time taken to return a response from the cache.
- **AI Cache Embeddings Latency**: Time taken to generate embedding during the cache.
{% endif_version %}

AI metrics are disabled by default as it may create high cardinality of metrics and may
cause performance issues. To enable them, set `ai_metrics` to true in the Prometheus plugin configuration.

Here is an example of output you could expect from the `/metrics` endpoint:

```bash
curl -i http://localhost:8001/metrics
```

Response:
```sh
HTTP/1.1 200 OK
Server: openresty/1.15.8.3
Date: Tue, 7 Jun 2020 16:35:40 GMT
Content-Type: text/plain; charset=UTF-8
Transfer-Encoding: chunked
Connection: keep-alive
Access-Control-Allow-Origin: *

{% if_version gte:3.0.x %}
# HELP ai_llm_requests_total AI requests total per ai_provider in Kong
# TYPE ai_llm_requests_total counter
ai_llm_requests_total{ai_provider="provider1",ai_model="model1",cache_status="hit",vector_db="redis",embeddings_provider="openai",embeddings_model="text-embedding-3-large",workspace="workspace1"} 100
# HELP ai_llm_cost_total AI requests cost per ai_provider/cache in Kong
# TYPE ai_llm_cost_total counter
ai_llm_cost_total{ai_provider="provider1",ai_model="model1",cache_status="hit",vector_db="redis",embeddings_provider="openai",embeddings_model="text-embedding-3-large",workspace="workspace1"} 50
# HELP ai_llm_provider_latency AI latencies per ai_provider in Kong
# TYPE ai_llm_provider_latency bucket
ai_llm_provider_latency_ms_bucket{ai_provider="provider1",ai_model="model1",cache_status="",vector_db="",embeddings_provider="",embeddings_model="",workspace="workspace1",le="+Inf"} 2
# HELP ai_llm_tokens_total AI tokens total per ai_provider/cache in Kong
# TYPE ai_llm_tokens_total counter
ai_llm_tokens_total{ai_provider="provider1",ai_model="model1",cache_status="",vector_db="",embeddings_provider="",embeddings_model="",token_type="prompt_tokens",workspace="workspace1"} 1000
ai_llm_tokens_total{ai_provider="provider1",ai_model="model1",cache_status="",vector_db="",embeddings_provider="",embeddings_model="",token_type="completion_tokens",workspace="workspace1"} 2000
ai_llm_tokens_total{ai_provider="provider1",ai_model="model1",cache_status="hit",vector_db="redis",embeddings_provider="openai",embeddings_model="text-embedding-3-large",token_type="total_tokens",workspace="workspace1"} 3000
# HELP ai_cache_fetch_latency AI cache latencies per ai_provider/database in Kong
# TYPE ai_cache_fetch_latency bucket
ai_cache_fetch_latency{ai_provider="provider1",ai_model="model1",cache_status="hit",vector_db="redis",embeddings_provider="openai",embeddings_model="text-embedding-3-large",workspace="workspace1",le="+Inf"} 2
# HELP ai_cache_embeddings_latency AI cache latencies per ai_provider/database in Kong
# TYPE ai_cache_embeddings_latency bucket
ai_cache_embeddings_latency{ai_provider="provider1",ai_model="model1",cache_status="hit",vector_db="redis",embeddings_provider="openai",embeddings_model="text-embedding-3-large",workspace="workspace1",le="+Inf"} 2
# HELP ai_llm_provider_latency AI cache latencies per ai_provider/database in Kong
# TYPE ai_llm_provider_latency bucket
ai_llm_provider_latency{ai_provider="provider1",ai_model="model1",cache_status="hit",vector_db="redis",embeddings_provider="openai",embeddings_model="text-embedding-3-large",workspace="workspace1",le="+Inf"} 2
{% endif_version %}
```

{:.note}
> **Note:** If you don't use any cache plugins, then `cache_status`, `vector_db`,
`embeddings_provider`, and `embeddings_model` values will be empty. 

## Accessing the metrics

In most configurations, the Kong Admin API will be behind a firewall or would
need to be set up to require authentication. Here are a couple of options to
allow access to the `/metrics` endpoint to Prometheus:


1. If the [Status API](/gateway/latest/reference/configuration/#status_listen)
   is enabled, then its `/metrics` endpoint can be used.
   This is the preferred method.

1. The `/metrics` endpoint is also available on the Admin API, which can be used
   if the Status API is not enabled. Note that this endpoint is unavailable
   when [RBAC](/gateway/api/admin-ee/latest/#/operations/get-rbac-users) is enabled on the
   Admin API (Prometheus does not support Key-Auth to pass the token).


