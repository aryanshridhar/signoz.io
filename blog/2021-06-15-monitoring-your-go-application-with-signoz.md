---
title: How to set up Golang application performance monitoring with open source monitoring tool - SigNoz
slug: monitoring-your-go-application-with-signoz
date: 2021-06-15
tags: [application-monitoring, go-monitoring, opentelemetry]
authors: ankit_anand
description: In this article, learn how to setup application monitoring for Golang apps using an open-source solution, SigNoz.
image: /img/blog/2021/06/golang_app_monitoring_cover_hc.webp
keywords:
  - go application monitoring
  - opentelemetry
  - opentelemetry go
  - go app
  - golang
  - distributed tracing
---

In this article, learn how to setup application monitoring for Golang apps using an open-source solution, SigNoz.

<!--truncate-->

![Cover Image](/img/blog/2021/06/golang_app_monitoring_cover_hc.webp)

If you want to check our Github repo before diving in 👇

[![SigNoz GitHub repo](/img/blog/common/signoz_github.webp)](https://github.com/SigNoz/signoz)

Scalability, Reliability, Maintainability...

The list goes on for the benefits of microservices architecture in today's world. But along with these benefits also comes the challenges of complexity. How do you ensure your distributed infrastructure, which spans across servers, datastores, cloud vendors, and third-party APIs, is in fine health to meet customer requirements all the time?

A single user request may get routed through three, five, eighteen, or hundred different layers of services.

And it quickly becomes unrealistic for teams to identify which service was responsible for slowing a request down. Engineering teams need a system that brings context to this complexity. A system which enables quick identification of potential issues so that it can be resolved as quickly. And that's where there is a need for a robust monitoring framework.

### Table of contents

- [Introducing SigNoz](#introducing-signoz)
- [Installing SigNoz](#part-1-installing-signoz)
- [Instrumenting sample Golang app](#part-2-instrumenting-sample-golang-app)
- [Using SigNoz Dashboard to identify issues causing high latency](#using-signoz-dashboard-to-identify-issues-causing-high-latency-in-your-app)

## Introducing SigNoz

SigNoz is a full-stack open-source application monitoring and observability platform which can be installed within your infra. You can track metrics like p99 latency, error rates for your services, external API calls, and individual endpoints. With service maps, you can quickly assess the health of your services.

import Screenshot from "@theme/Screenshot"

<Screenshot
   alt="Service Maps on SigNoz dashboard"
   height={500}
   src="/img/blog/2021/06/ezgif.com-gif-maker-1.gif"
   title="Service maps on SigNoz dashboard"
   width={700}
/>

And once you know the affected service, trace data can help you identify the exact code causing the issue. Using SigNoz dashboard, you can visualize your traces easily with flamegraphs.

<Screenshot
   alt="Flamegraphs on SigNoz dashboard"
   height={500}
   src="/img/blog/2021/06/screenzy-1622399034895.webp"
   title="Distributed tracing visualized with flamegraphs on SigNoz dashboard"
   width={700}
/>

Now let's get down to some action and see everything for yourself.

We will divide the tutorial into two parts:

1. Installing SigNoz
2. Instrumenting sample app to start monitoring

## Part 1 - Installing SigNoz

### Installing SigNoz

You can get started with SigNoz using just three commands at your terminal.

```jsx
git clone https://github.com/SigNoz/signoz.git
cd signoz/deploy/
./install.sh
```
<br></br>

For detailed instructions, you can visit our documentation.

[![Deployment Docs](/img/blog/common/deploy_docker_documentation.webp)](https://signoz.io/docs/deployment/docker/?utm_source=blog&utm_medium=opentelemetry_flask)

When you are done installing SigNoz, you can access the UI at: [http://localhost:3000](http://localhost:3000/application)

The application list shown in the dashboard is from a sample app called HOT R.O.D that comes bundled with the SigNoz installation package.

<Screenshot
   alt="SigNoz dashboard showing application list"
   height={500}
   src="/img/blog/2021/08/signoz_dashboard_hc.webp"
   title="SigNoz Dashboard"
   width={700}
/>

Now that you have SigNoz up and running, let's see how instrumentation works. Instrumentation is the process of implementing code instructions to monitor your application's performance. Instrumentation is key to see how your application handles the real world. It helps you generate trace data which you can then use to understand what's happening inside your systems.

SigNoz supports [OpenTelemetry](https://opentelemetry.io/) as the primary way for users to instrument their application. OpenTelemetry is a single, vendor-agnostic instrumentation library with support for both automatic and manual instrumentation. More details on OpenTelemetry Golang SDKs and APIs [here](https://github.com/open-telemetry/opentelemetry-go).

## Part 2 - Instrumenting sample Golang app

To see how SigNoz can start reporting data of a Golang app, let's see how it works with a sample bookstore app ([GitHub repo](https://github.com/SigNoz/sample-golang-app)).

It is a simple bookstore app with a REST API that provides book data and performs CRUD operations. The app uses Gin framework to build a RESTful API. Gin is a high-performance HTTP web framework written in Golang containing a set of commonly used functionalities like routing, middleware support and rendering.

OpenTelemetry has specific instrumentation packages to support popular Golang packages and use cases.  For example, this app uses the Gin framework for request routing. OpenTelemetry provides instrumentation package named **otelgin** to instrument the Gin framework which you need to import in your app. You can find the complete list of supported Golang packages [here](https://github.com/open-telemetry/opentelemetry-go-contrib/tree/main/instrumentation).

**Prerequisites**

Verify if you have Golang installed on your machine by running `$ go version` on your terminal. If you don't have Golang installed, you can download it [here](https://golang.org/doc/install).

### Steps to instrument Golang app

1.  Clone sample Golang app repository<br></br>
    From your terminal use the following command to clone sample Golang app GitHub repository.

    ```
    git clone https://github.com/SigNoz/sample-golang-app.git
    ```

2.  Update path to **sample-golang-app** & check if the app is working<br></br>
    Update your terminal path to the sample app directory and check if the app is working or not using the following command:

        cd sample-golang-app
        go run main.go
    This runs the gin application at port `8090`. Try accessing API at `http://localhost:8090/books`

    If you see an empty array, it means your application is working. You can check out how to write, update and delete books in your array from the article [here](https://blog.logrocket.com/how-to-build-a-rest-api-with-golang-using-gin-and-gorm/).

    <Screenshot
    alt="endpoint of bookstore app"
    height={500}
    src="/img/blog/2021/06/screenzy-1623261415095.webp"
    title="endpoint of our bookstore app"
    width={700}
    />

3.  Once you ensure that your application is working, exit the server by pressing `Ctrl + C` on your mac terminal.

4.  **Set up OpenTelemetry Golang instrumentation library**<br></br>
    The file `main.go` has instructions to import all the necessary OpenTelemetry packages in order to instrument the sample app. For this app, we import the following OpenTelemetry packages.

    ```
    import (
    "go.opentelemetry.io/contrib/instrumentation/github.com/gin-gonic/gin/otelgin"
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/exporters/otlp"
    "go.opentelemetry.io/otel/exporters/otlp/otlpgrpc"
    "go.opentelemetry.io/otel/label"

    "go.opentelemetry.io/otel/sdk/resource"
    sdktrace "go.opentelemetry.io/otel/sdk/trace"
    )
    ```

    You just need to run it with the necessary environment variables in order to start sending data to SigNoz. Use the following command to run and configure the app to send data to SigNoz:

    ```
    SERVICE_NAME=goApp INSECURE_MODE=true OTEL_METRICS_EXPORTER=none OTEL_EXPORTER_OTLP_ENDPOINT=<IP of SigNoz backend>:4317 go run main.go
    ```

    `Ip of SigNoz` can be replaced with `localhost` in this case. Hence, the final command becomes:

    ```
    SERVICE_NAME=goApp INSECURE_MODE=true OTEL_METRICS_EXPORTER=none OTEL_EXPORTER_OTLP_ENDPOINT=localhost:4317 go run main.go
    ```

And, congratulations! You have instrumented your sample Golang app. Hit the `/books` endpoint of the bookstore app at [http://localhost:8090/books](http://localhost:8090/books). Refresh it a bunch of times in order to generate load, and wait for 1-2 mins for data to appear on SigNoz dashboard.

You can now access the SigNoz dashboard at [http://localhost:3000](http://localhost:3000/) to monitor your app for performance metrics.

<Screenshot
   alt="Sample golang app in the list of applications"
   height={500}
   src="/img/blog/2021/06/signoz_ui_goapp.webp"
   title="Sample Golang app appears in the list of application"
   width={700}
/>

If you want to instrument your own Golang application, you can read about it in our [documentation](https://signoz.io/docs/instrumentation/golang/#instrumentation-of-a-sample-golang-application).

## Using SigNoz dashboard to identify issues causing high latency in your app

Now that you have installed SigNoz, let's see how you can identify specific events causing high latency in your deployed applications.

In just 5 easy steps, our dashboard lets you drill down to events causing a delay in your deployed apps 👇

1. **Choose the service you want to inspect**

   ![List of services monitored](/img/blog/2021/06/dashboard_applications_list-2.webp)

2. **Choose the timestamp where latency is high and click on view traces**

   ![Dashboard showing RED metrics](/img/blog/2021/06/dashboard_view_traces-1.webp)

3. **Choose the trace ID with the highest latency**

   ![See list of traces](/img/blog/2021/06/dashboard_highest_traceid.webp)

4. **Inspect distributed traces with flamegraph**

   ![Flamegraphs for distributed tracing](/img/blog/2021/06/dashboard_flamegraph.webp)

5. **Zero in on the highest latency event and take action**

   ![Zoom in to specific spans](/img/blog/2021/06/dashboard_highest_latency.webp)

If you need any help with trying out SigNoz, feel free to mail me at ankit.anand@signoz.io.

Check out our [documentation](https://signoz.io/docs/deployment/docker) for more installation guides and troubleshooting instructions.

They say, "If it's not monitored, then it's not in production." And with SigNoz you can start monitoring your applications now. Enabling your team to resolve issues quickly in production is critical to maintaining complex distributed systems in fine health.

At SigNoz, we are committed to making the best open-source, self-hosted tool for application performance monitoring. Feel free to check out our GitHub repo here:

[![SigNoz GitHub repo](/img/blog/common/signoz_github.webp)](https://github.com/SigNoz/signoz)
