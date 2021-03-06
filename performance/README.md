# :electric_plug: Apache OpenWhisk - Performance Tests
A few simple but efficient test suites for determining the maximum throughput and end-user latency of the Apache OpenWhisk system.

## Workflow
- A standard OpenWhisk system is deployed. (_Note that the API Gateway is currently left out for the tests._)
- All limits are set to 999999, which in our current use case means "No throttling at all".
- The deployment is using the docker setup proposed by the OpenWhisk development team: `overlay` driver and HTTP API enabled via a UNIX port.

The load is driven by the blazingly fast [`wrk`](https://github.com/wg/wrk).

#### Travis Machine Setup
The [machine provided by Travis](https://docs.travis-ci.com/user/ci-environment/#Virtualization-environments) has ~2 CPU cores (likely shared through virtualization) and 7.5GB memory.

## Suites

### wrk

#### Latency Test
Determines the end-to-end latency a user experience when doing a blocking invocation. The action used is a no-op so the numbers returned are the plain overhead of the OpenWhisk system. The requests are directly against the controller.

- 1 HTTP request at a time (concurrency: 1)
- You can specify how long this test will run. Default are 30s.
- no-op action

**Note:** The throughput number has a 100% correlation with the latency in this case. This test does not serve to determine the maximum throughput of the system.

#### Throughput Test
Determines the maximum throughput a user can get out of the system while using a single action. The action used is a no-op, so the numbers are plain OpenWhisk overhead. Note that the throughput does not directly correlate to end-to-end latency here, as the system does more processing in the background as it shows to the user in a blocking invocation. The requests are directly against the controller.

- 4 HTTP requests at a time (concurrency: 4) (using CPU cores * 2 to exploit some buffering)
- 10.000 samples with a single user
- no-op action

#### Running tests against your own system is simple too!
All you have to do is use the corresponding script located in `/*_tests` folder, remembering that the parameters are defined inline.

### gatling

#### Simulations

You can specify two thresholds for the simulations.
The reason is, that Gatling is able to handle each assertion as a JUnit test.
On using CI/CD pipelines (e.g. Jenkins) you will be able to set a threshold on an amount of failed testcases to mark the build as stable, unstable and failed.

##### ApiV1Simulation

This Simulation calls the `api/v1`.
You can specify the endpoint, the amount of connections against the backend and the duration of this burst.

Available environment variables:

```
OPENWHISK_HOST          (required)
CONNECTIONS             (required)
SECONDS                 (default: 10)
REQUESTS_PER_SEC        (required)
MIN_REQUESTS_PER_SEC    (default: REQUESTS_PER_SEC)
```

You can run the simulation with (in OPENWHISK_HOME)
```
OPENWHISK_HOST="openwhisk.mydomain.com" CONNECTIONS="10" REQUESTS_PER_SEC="50" ./gradlew gatlingRun-ApiV1Simulation
```
