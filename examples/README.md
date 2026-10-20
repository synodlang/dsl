# Examples

Canonical sample programs. They form one module,
`github.com/synodlang/examples`, and move to the `examples` repository
once the organization exists. Each behaviour package has a `demo/`
program next to it.

| Folder | Topic | What to watch | Run |
| --- | --- | --- | --- |
| `tcp` | TCP three-way handshake | SYN, SYN-ACK, ACK and the state changes on both ends | `synod run ./tcp/demo` |
| `gossip` | Gossip membership and failure detection | Heartbeats spreading; a crashed node marked down after 4s; generations after restart | `synod run ./gossip/demo` |
| `paxos` | Single-decree Paxos (the Synod protocol) | Two racing proposers; the higher ballot adopts any accepted value; the invariant "at most one chosen value" | `synod run ./paxos/demo` |
| `raft` + `raftlook` | Raft leader election | Randomized timeouts, votes, a leader crash, a new term; "never two leaders in a term" | `synod run ./raft/demo` |
| `twopc` | Two-phase commit | The blocking problem when the coordinator crashes between phases; the atomicity invariant | `synod run ./twopc/demo` |
| `ring` + `ringlook` | Consistent hashing with virtual nodes | Which keys move when a fifth node joins | `synod run ./ring/demo` |
| `cassandra` | A Cassandra-style cluster: gossip + ring + replication factor 3 + QUORUM + hinted handoff | A write succeeding with a replica down, and the hint replayed after restart | `synod run ./cassandra/demo` |
| `webapp` | Cache-aside behind a load balancer (`api`, `lb`, `app`, `cache`, `db`) | Hit rate warming up; what happens when the cache crashes and there is no timeout | `synod run ./webapp/demo` |
| `wal` | Write-ahead log and checkpoint recovery | Which writes survive a crash between checkpoints | `synod run ./wal/demo` |

## Things to try (Break it)

- **Paxos:** delete the loop that adopts the highest prior value, run
  seeds 1–200, and watch `always` break.
- **Raft:** shorten the election timeout range to `150ms and 160ms` and
  watch split votes.
- **Two-phase commit:** move the coordinator crash to `after 80ms`; the
  decision now reaches the participants.
- **Cassandra:** set `Quorum` to 3; the second write no longer succeeds
  while a replica is down.
- **Web app:** add a timeout in `app` so requests fail fast when the cache is down.

## Notes for implementers

- These programs are the acceptance tests of `implementation/10-roadmap.md`.
- Timings in the demos (for example the 45ms coordinator crash) were
  chosen to land between protocol phases with typical latencies. If the
  golden trace for a seed does not show the intended story, adjust the
  timing or the seed, never the protocol, and record it here.
