# Architecture

## Threading/Runtimes

Ztunnel runs two distinct async runtimes:
* The "main" thread, runs a single threaded Tokio runtime for admin purposes, such as debug interfaces and XDS. This is isolated to avoid impacting the data plane.
* The "worker" thread(s) run a multi-thread Tokio runtime to handle users requests. This defaults to 2 threads, but is configurable.

## Ports

Ztunnel runs with the following ports:

| Port  | Purpose                               |
|-------|---------------------------------------|
| 15001 | Pod outbound traffic capture          |
| 15006 | Pod inbound plaintext traffic capture |
| 15008 | Pod inbound HBONE traffic capture     |
| 15080 | Pod outbound `socks5` traffic         |
| 15021 | Readiness                             |
| 15000 | Admin (Admin thread) (Localhost)      |
| 15020 | Metrics (Admin thread)                |

The three admin ports (Readiness, Admin, and Metrics) are intentionally split.

* The readiness port ought to run on the "main" thread to ensure we are actually checking the path the data plan handles
* The admin port must be only on localhost, and it should be on the admin thread for isolation
* The metrics port should be on the admin thread to avoid isolation.
  This *could* be on the readiness port, but historically we had found that the stats query can be very expensive and lead to tail latencies in the data plane.