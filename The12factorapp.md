# The 12 factor app



Taken from [https://12factor.net](https://12factor.net/), the 12 factor app is a great set of recommendations for creating web apps, or SAAS apps. I personally think it is a great way to create any app, from arranging ones own used python apps to python libraries to data-science and mlops apps such as gonal and ortho

## Principles

- using **declarative formats** for setup automation
- **clean contract** with underlying operating system
- Make it easily deployable on **modern cloud platforms** which will help for on-prem deployment. I will add that it might be worth making it easily deployable on **serverless platforms**.
- Minimize **divergence** from dev to production. I actually somewhat disagree with this one, and have an entire catalog lookup system in gonal built around this divergence. My notion is that serverless-useful local-first platforms such as SQLite and duckdb are to be preferred. The user should not have to run a server. Similarly I prefer command line automations to observability to web based observability.
- Being able to **scale** up without significant changes to tooling, architecture, or development practices. One way to think about this is to scale from local to ci/cd to prod. Also in the situation of mlops this scale means going to multiple cpus/gpus/high-availability/load-balancing from the usual dev/ci-cd case

## A listing of the 12 factors

Ok now lets go over the 12 factors one-by-one. In this section we’ll just list them so that we can see them at a glance.

1. Codebase: one codebase tracked in revision control with many deploys
2. Dependencies: explicitly declare and isolate dependencies
3. Config: store config in the environment
4. Backing services: treat backing services as attached resources
5. Build, release, run: strictly separate build and run stages
6. Processes: execute the app as one or more stateless processes
7. Port binding: export services via port binding
8. Concurrency: scale out via the process model
9. Disposability: maximize robustness with fast startup and graceful shutdown
10. Dev/prod parity: keep development, staging, and production as similar as possible
11. Logs: treat logs as event streams
12. Admin processes: run admin/management tasks as one-off processes.

## One Codebase

*one codebase in revision control, many deploys*

- track your codebase in version control
- The app should be in one codebase. Thus if you need common code consider making libraries in separate codebase which are utilized by this app
- These libraries will be included via the dependency manager: for example, pixi, toml, etc.
- There are multiple types of deploys for the app. Different branches on deploys may be used in development and ci/cd to test things, but the deploy of any branch of the app to local, ci/cd, or prod should only differ in resources

## Dependencies

*explicitly declare and isolate dependencies*

- a twelve factor app never relies on implicit existence of system-wide packages. All dependencies must be declared explicitly (“*dependency declaration*”) via the manifest or lock file: conda.lock, poetry.lock, or pixi.lock/cargo.lock.
- Furthermore one must provide for *dependency isolation*: pipenv or conda envs or virtual end do this. `pixi` can again be reused for this purpose
- This means that only the tools to do this stuff are needed, and can be installed globally or in a dev environment. We prefer non-python tools such as pixi, uv, rye and ruff, and dont specify whether dev tools should be in a dev env or globa; Consider that you might want to build a devcontainer from these, no matter how you do it.
- If one needs to shell out to a tool such as `gh` or `git`, make sure these are *vendored* into your app
- The logical limit of this is docker containers and devcontainers.
- If one is using custom libraries there needs to be a build part in them. My thought is to use `hatchling` for this for python builds or local builds. If you are using a per folder build tool such as pixi you can do a dev build for this purpose, especially during development but note them that you wont get the exact versioning you need. Better to build wheels/conda in ci/cd and obtain from a OCI compliant package registry as binaries/source.
- The OCI registry may be used for docker images as well.

## Configuration

*store config in the environment*

- the config refers to stuff that changes during deploys and development, and which should not be hardcoded into code: this includes paths, resources for other servers, host names in a deploy, etc
- Configs should not be stored in code, but rather come from config files which are not part of the repository, and even more so from environment variables that can be set in a `run.sh` or in a `.env` file. These must be gitignored but there is a danger to this, of-course. Configs might be obtained from a catalog, but security is key: see below.
- It also includes **secrets** such as passwords and AWS credentials. While these could be passed via the env or stored in dot env files, and will likely be done so in local dev, using a secrets application which encrypts and decrypts is a good idea. This could be implemented as part of a catalog using a AES or better key..
- The 12factor app website suggests not to group and namespace commits, but I disagree: readability is far more important, and it is good to do it via environment such as dev. The worry about config explosion is not true if stuff is namespaces under individual developers.

## Backing Services

*treat backing services as attached resources*

- a backing service, such as a database, or queue, or api, is one consumed over the network. One question one must answer is whether backing services are serverless, or long-term servered, or even local file system based as is convenient during dev
- These are treated as resources by the application and their location and port discovered using config or catalog
- Because these are treated as resources, the coupling of the app to them is loose, and you should structure your app to not worry about the order of boot up, and to gracefully fail if resources are not found
- The 12factor.app site treats a shared MySQL as 2 resources but i am not convinced that this is the right idea. The scaling of a database or backend resource should be removed from the app. This makes the app simpler but less monolithic. A backend like kubernetes could take care of other things like load balancing
- In my thoughts we dont only represent these as resources: but even better we represent these as cataloged resources with namespaces, and we can add regular reads and writes to that

## Build, Release, Run

*Strictly separate build and run stages*

![](https://12factor.net/images/release.png)
- the **build stage** transforms a code repo into an executable bundle known as a build: for stuff like rust cargo will fetch dependencies and compile. For interpreted languages the build stage will fetch the dependencies and get the app ready from deployment for the repo
- Herein lies a deep question: we could be building a library or a SAAS, and then a “build” stage makes a lot of sense. But what if it is a user application, even in prod, like a ml pipeline. What does build look like then? It might just be the running of a particular piece of code, like a training loop.
- The **release stage** takes the build produced by the build stage and combines it with the appropriate dev/ci/prod config to make a release that is ready for execution. Releases should have unique release-ids.
- The release stage is often done via the CD process, and running is sometimes done as part of that process. This is usually done from something like github actions.
- The **run stage** or **runtime** runs the selected release in the appropriate environment. Runs can happen independent of releases.
- 12factor apps want to create a strict separation between these situations. For example `Capistrano` symlinks a current to the present release. 12factor also demands that we don’t demonize: but rather use something like systems or procman: how do we do this in user space, making sure logs are properly transported

## Processes

*Execute the app as one or more stateless processes*

- we might be executing locally or otherwise..indeed in prod we might have multiple processes, or be connecting to running processes such as databases
- 12 factor processes are stateless and share-nothing…all state must be stored in a stateful backing processes such as a database. One must assume all local state is wiped out on a crash or restart
- Assuming that the local file system is cleaned out has some consequences. You can’t rely on web server or other assets being there on the local file system. This has consequences in checkpointing or intermediate output systems: if you need to startup a process again it makes sense to store intermediate processes in the warehouse
- This also means that asset compilation, such as images/css/etc must be done during the build stage. Don’t assume you will find these on the filesystem.
- Thus time-expired cookies on the rest should also come from systems such as Reid’s or memcached, with the ability to persist if necessary

## Port Binding

*Export services using port binding*

- The 12-factor app should be completely self contained..this means that the web should be exported by a local port. A proxy web server can connect to this: a routing layer in prod.
- any other service can also be exported via port binding
- Any one app can then become a binding service for another app. One can then question how apps should be started or not-started together…but it makes sense that any separate microservice be a separate-self-contained app, with its own admin and stuff…eg a database can be reconstituted from backup and populated and all that

## Concurrency

*Scale out via a process model*

![](https://12factor.net/images/process-types.png)
- 12factor apps are microservice in the sense that each type of work is assigned to a process type: http for web, and long handled background tasks off a process queue..this is a variation on the Unix process model..
- Each process can handle its own threads or asynchrony loops if appropriate, but for horizontal scaling you want to be able to add similar processes
- This is due to the shared nothing horizontally partitionable nature of 12 factor processes..this means concurrency is easy
- However, this means that load-balancing becomes the responsibility of the routing layer. 
- It also asks that from our config, how does the notion of horizontal scaling work…initial configs might refer to multiple processes, or single processes…12 factor recommends the latter. But then how do you scale? This means that scaling must happen through something runtime like the catalog, and not just through the config. `Procfile` or `k8s` based tools can help with scaling- The port binding means that perhaps a range of ports must be reserved in some kind of port config at a inherent level: procfile tools will do this…
- 12 factor apps should never daemonize or write paid files. ;eave it to systems or foreman and let them handle output streams, respond to crashed processes, and handle user initiated restarts or shutdowns. The exception to this is in the dev stage…systems can be run in user space for example, but ideally we’d want the ability for this “orchestrator” to run containers as well
- Apps like overmind and hive mind are `Procfile` based, just like foreman and provide good simple-process orchestrators. I am not sure how logs can be multiplexed to observability tools.
- Such orchestrators could be things like dagster as well which will take care of the orchestration of their processes, Dag running, and log transport as well..and with cluster managers and runners such as ray run across a cluster and in parallel as well.

## Disposability

*apps should be disposable, which means they could be started or stopped at moments notice*

- startup and stop speed is important, even if you are running on firecracker or a docker container, because that is key to scaling
- Thus processes should shutdown gracefully when they receive a SIGTERM from the process manager…implement a SIGTERM handler or use a framework which has one
- For a long-running worker process, return the current job to the job queue. Thus also, long running ops need some idempotency, re-entrance, or DAG based job management

## Dev/Prod parity

*keep dev, staging, and prod as similar as possible*

- this is easier to do with a SAAS app than a machine learning app, and indeed
- I disagree with 12 factors on the use of the same backing resources: local resources are faster..still docker may be easily used to create production backend environments
- By the same token, using generic code though: unless a database is a critical part of the app and wont be replaced use a common sql dialect. For example, sqlite may be used in place of Postgres for a catalog or mlflow database, but duckdb can be a critical component of io systems..and in the latter use whatever non-standard syntax you need on the understanding that it will be used in prod too.

## Logs

*treat logs as event streams*

- logs are critical, prints and print to steer should go to log
- Orchestrators should handle this for you, but you literally may want to `tee` off to an observability system as well or use a queue
- A 12 factor app should never concern itself with the rating or storage of the log stream. Instead write to `stdout`, unbuffered
- Tux can be used to connect to the stream but the stream may also be multiplexed in `staging` or `prod` by the orchestrator, combined with other streams, and routed to one or more final destinations..`logplex` and `fluentd` might help
- They can also be sent to plunk or Kafka and from there to hive or other stores where they might be analyzed, or to things like plunk where immediate analysis and active alerting is possible

## Admin Processes

*run admin/management as 1-off CLI tasks*

- migration of databases, population, one-off scripts
- Clips to add data to a catalog, run a training, etc
- Pixi’s run commands are good examples of this: they will help make sure that the script environment is the same as the app environment
- These might need to be ssh-able to transportable by a `k3s` like orchestrator
- Running a repl configured for the app might be useful, although we should take care to run in a read-only mode. 