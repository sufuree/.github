# Sufuree

Sufuree is an experimental runtime that wakes isolated applications on demand
and puts them back to sleep when they're idle.

Each application runs in its own lightweight virtual machine, not a shared-kernel
container. When a request arrives for an idle application, Sufuree wakes it,
routes the request to it, and later drains and stops it so that idle
applications don't use compute.

## Status

Sufuree is in early design. No code is ready to use yet.

We're working from a draft RFC and an architecture review of it. We're
not building the full platform up front. Each part has to pass its own test
before we build on it:

1. Pick one real, bursty, stateless HTTP workload and record what it needs:
   traffic, dependencies, and latency.
2. Qualify one guest and VMM combination on a fixed host. Measure cold and warm
   time to first response at p50, p95, and p99.
3. Test the hard cases: simultaneous requests through two ingress instances,
   wake timeouts, client cancellation, drain races, coordinator restarts, host
   loss, and stale commands.
4. Measure the full cost of running it while idle and while active, including
   storage, networking, backups, and operator time.
5. Adopt it only if the workload meets its latency and recovery targets and the
   cost or isolation benefit is worth running another system.

We'll publish measurements once we've taken them. Until then, we don't make
claims about latency or cost.

## Design principles

- **One owner per lifecycle.** Each placement has a single lifecycle owner and a
  generation number that only goes up. Hosts reject commands from stale owners.
  A gossiped route is a cache. It never authorizes a stop.
- **Infrastructure stays with its owner.** Sufuree manages guests on capacity
  it's assigned. It doesn't create or delete cloud hosts.
- **No hidden state in sleeping guests.** The first version keeps application
  data in existing managed services. Stateful guests need a stated recovery
  point objective (RPO) and recovery time objective (RTO), and a power-loss
  test, before we support them.
- **Bounded admission.** Before waking anything, Sufuree enforces limits on
  request size, queue depth, deadlines, and per-application concurrency. It
  doesn't buffer requests forever or retry requests that aren't idempotent.
- **Unknown stays unknown.** If Sufuree can't confirm a host's state, it doesn't
  start a second writable copy of the guest.

## Not in scope yet

Multicloud support, distributed consensus, stateful snapshots, WebSocket offload,
and a hosted backend-as-a-service layer are deferred. We'll add each one only
when a real workload needs it.

## Repositories

- [`sufuree`](https://github.com/sufuree/sufuree): the runtime, design documents,
  and research.
