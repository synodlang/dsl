# File formats

All JSON is UTF-8 with keys in the order shown. Times are integers:
`t` in microseconds of simulated time, `at` in milliseconds of
presentation time. Every file starts with a `format` version; readers
reject unknown major versions.

## synod.mod

```
module github.com/synodlang/examples
synod 0.2
require github.com/someone/raft v1.2.0
```

One directive per line: `module` (once), `synod` (language version,
once), `require` (zero or more, sorted). `synod.sum` holds one line per
dependency: `<module> <version> <commit sha> <tree hash>`.

## trace.jsonl

One JSON object per line. Line 1 is the header:

```json
{"format":"synod-trace/1","synod":"0.2","seed":7,"program":"examples/paxos","packages":["paxos","std/net","std/sim"]}
```

Then one line per trace event:

```json
{"seq":12,"t":20431,"ev":"net.Sent","node":"n1","msg":{"id":5,"type":"paxos.Prepare","fields":{"number":101}},"from":"n1","to":"n3"}
{"seq":13,"t":31877,"ev":"net.Arrived","node":"n3","msg":{"id":5,"type":"paxos.Prepare","fields":{"number":101}},"from":"n1","to":"n3"}
{"seq":14,"t":31877,"ev":"sim.Changed","node":"n3","field":"paxos.Promised","old":0,"new":101}
{"seq":20,"t":52010,"ev":"sim.Emitted","node":"n2","record":{"type":"paxos.Learned","fields":{"Value":"x"}}}
{"seq":31,"t":90000,"ev":"sim.Broken","rule":"paxos/paxos.synod:109: always ((lists.Size ChosenValues) at most 1)"}
```

| `ev` | Extra keys |
| --- | --- |
| `sim.Started` | none |
| `sim.Ended` | `reason`: `"empty"`, `"stop"`, `"limit"` |
| `sim.Added`, `sim.Restarted` | `node`, `runs` (packages), `state` (full exported state, `{ "pkg.Field": value }`) |
| `sim.Removed`, `sim.Crashed` | `node` |
| `sim.Changed` | `node`, `field`, `old`, `new` |
| `sim.Injected` | `node`, `record` |
| `sim.Fired` | `node` |
| `sim.Broken` | `rule` |
| `sim.Emitted` | `node` (`null` when emitted by `sim.Nobody` from main), `record` |
| `sim.Failed` | `code`, `message`, `pos` |
| `net.Sent`, `net.Arrived` | `node`, `msg`, `from`, `to` |
| `net.Dropped` | `node`, `msg`, `from`, `to`, `reason` |
| `net.Partitioned` | `a`, `b` (lists of node names) |
| `net.Healed` | none |
| `disk.Written` | `node`, `key` |

Value encoding: text → JSON string; number → JSON number; bool → JSON
bool; time → `{"time":20000}` (µs); node → `{"node":"n3"}`; list → array;
map → `{"map":[[k, v], …]}` in key order; record → `{"type":…,"fields":{…}}`.
Only exported fields are written for records and state; a record's
unexported fields appear as `"…"` so looks cannot depend on them.

## scene.json

Read by the `player` library (the Mac app and the site both use it).

```json
{
  "format": "synod-scene/1",
  "duration": 18450,
  "canvas": {"w": 1600, "h": 1000},
  "theme": {"Paper": "#151A2E", "Ink": "#EDE9E1", "Line": "#3A4366", "Quiet": "#8A90A8",
            "Accent": "#F4A340", "Good": "#5CC8B8", "Warning": "#E8C35A", "Critical": "#E86F5A"},
  "shapes": [
    {"id": "s7", "kind": "rect", "props": {"w": 96, "h": 56, "radius": 12},
     "keys": [
       {"at": 0,    "x": 800, "y": 300, "opacity": 0, "fill": "Ink"},
       {"at": 300,  "opacity": 1},
       {"at": 5200, "fill": "Accent"}
     ]}
  ],
  "markers": [
    {"at": 4100, "kind": "pause", "text": "Who becomes leader next?"},
    {"at": 6000, "kind": "caption", "text": "Raft never has two leaders in one term."},
    {"at": 9000, "kind": "broken", "text": "always ((lists.Size ChosenValues) at most 1)"}
  ],
  "trace": [ {"seq": 12, "at": 1240} ]
}
```

- `kind`: `circle`, `rect`, `label`, `line`, `group`. `line` has `from`/`to` shape ids.
- Each keyframe sets only the properties that change. Between two keys
  the player interpolates with the house curve (smootherstep); colors
  interpolate in OKLab. Colors are theme names, never raw hex.
- `trace` maps each trace `seq` to its presentation slot, so the player can
  show "simulated time" and jump to a fact.

## check.json (`synod check --json`)

```json
{"format":"synod-check/1","ok":false,"diagnostics":[
  {"file":"gossip/gossip.synod","line":14,"col":3,"severity":"error","code":"unknown-phrase",
   "message":"no phrase matches \"net.Sned digest from n to peer\"","hint":"did you mean \"net.Send [m: msg] from [a: node] to [b: node]\"?"}
]}
```

`synod check --json --tokens` adds, per file, a list of tokens with their
resolved role (`language`, `phrase`, `name`, `field`, `type`, `value`,
`comment`) and the definition location, for editor highlighting and
go-to-definition.
