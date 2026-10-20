# BCU Computing Society - Wireshark Incident Investigation

A self-contained network-forensics exercise: a real, validated packet
capture of a fictional attack against the BCU Computing Society's web
application (`bcucompsoc.com`), for you to investigate in Wireshark and
write up as a portfolio-style incident report.

> **Fictional scenario.** All IP addresses use IANA documentation
> ranges (RFC 5737: `203.0.113.0/24` and `198.51.100.0/24`) and
> `bcucompsoc.com` is used only as a label inside the simulated
> traffic. Nothing in this repo touches any real system.

## What's in this repo

```
.
├── README.md                  <- this file (scenario + instructions)
├── compsoc-incident.pcap      <- the capture - open this in Wireshark
```

## The briefing

> Suspicious activity has been detected against the society's web
> application. Initial observations indicate repeated authentication
> failures against an administrative interface. A packet capture
> covering the relevant period has been preserved.
>
> Investigate the traffic, identify the sequence of events, determine
> whether administrative access was obtained, and document your
> findings using packet-level evidence.

That's genuinely all you're told going in. Don't assume the first
suspicious thing you find is the whole story - an HTTP `200` response
is not, by itself, proof that something worked.

## Requirements

- [Wireshark](https://www.wireshark.org/download.html) (any recent
  version; filter syntax below is confirmed valid on 4.2.x and 3.6+).
- No other software, accounts, or installs needed.

## How to use this repo

1. Clone or fork it, then open the capture:
   ```bash
   wireshark compsoc-incident.pcap
   ```
2. Work through the stages below, in order, using Wireshark's display
   filters and "Follow → TCP Stream" feature.
3. Write up your findings (a template structure is suggested at the
   bottom of this file) - then, and only then, compare against
   `reference-report/incident-report-golden-standard.pdf`.

### New to Wireshark?

Three panes: **packet list** (top - one row per packet), **packet
details** (middle - expandable headers for the selected packet), and
**packet bytes** (bottom - raw hex/ASCII). Click a packet, then expand
its layers in the details pane to see how they map to the bytes below.

Type a filter into the bar at the top and press Enter - this **hides**
non-matching packets, it doesn't delete anything. Right-click any HTTP
packet → **Follow → TCP Stream** to reassemble a full, readable
request/response exchange - the single most useful technique for this
exercise.

## Investigation stages

**Stage A - Understand the traffic.** How many distinct IP addresses
are talking? Use *Statistics → Conversations* or `ip.addr` filters to
list them. Which protocols appear? Find a few requests that look like
ordinary browsing - which host(s), and at what pace?

**Stage B - Reconnaissance.** One source address requests a run of
paths in quick succession, several of which don't exist. Which
address? List the paths and response codes. Why does the pattern look
automated rather than like a person browsing? Does one request reveal
where the real admin login lives?

**Stage C - Authentication activity.** Filter for requests to the
login endpoint you found in Stage B. How many attempts are there? What
response comes back each time - does **any** succeed? Where does this
activity stop, and for how long?

**Stage D - Keep going.** Don't assume the story's over when the
activity above stops. What's the very next request from the same
source, once things resume? How is it different from everything sent
before - not just the path, but the actual field values? What does the
response look like?

*(Hints, if you get stuck: compare the exact bytes of this request's
form data against the earlier attempts - something is different in
kind, not just a different guess. Then follow that TCP stream and read
the full response body, not just the status line.)*

**Stage E - Establish what happened.** Shortly after Stage D's
request, is there another request from the same source with a
similarly unusual value? What does the server respond with this time -
status code, and any notable response headers? Is there a subsequent
request to a path under `/admin/` carrying something in its headers
that none of the earlier requests had? What does the response contain,
and does it support the conclusion that this was treated as a logged-in
administrator? **Based on all of this: did the activity in Stage C, or
the activity in Stage D/E, actually obtain administrative access?**
Cite specific packets either way.

**Stage F - Impact and defences.** What does this capture *prove*,
with evidence? What does it *not* prove - what would server-side logs
add? Which controls (rate limiting, parameterised queries, WAF rules,
alerting, account lockout) would have prevented or detected each phase?
Why is plaintext HTTP acceptable in this teaching capture but not for a
real login system?

## Wireshark filter reference

| Filter | Purpose |
|---|---|
| `ip.addr == <address>` | All traffic to/from a host |
| `http` | Only HTTP requests/responses |
| `http.request` / `http.response` | Requests only / responses only |
| `http.request.method == "POST"` | Only POST requests |
| `http.request.uri == "<path>"` | Requests to a specific path |
| `http.response.code == 401` (or 403/404/302/500) | Responses with a given status |
| `http.set_cookie` | Responses that set a cookie |
| `http.cookie` | Requests that carry a cookie |
| `dns` | DNS queries/responses |
| `tcp.flags.syn==1 && tcp.flags.ack==0` | New connection attempts |

Menu tools worth knowing: **Follow → TCP Stream** (right-click a
packet), **Statistics → Conversations**, **Statistics → HTTP →
Requests**, **View → Time Display Format**.

## Suggested report structure

If you want to write this up for a GitHub portfolio, a report covering
the following sections - each claim backed by a packet number and
timestamp - works well:

1. Executive summary
2. Scope and methodology
3. Relevant hosts and network observations
4. Reconnaissance findings
5. Brute-force analysis
6. SQL injection evidence
7. Incident timeline
8. Root cause and impact assessment
9. Defensive recommendations
10. Limitations and unresolved questions
11. Evidence references (a table: timestamp / packet # / observation /
    interpretation / confidence)
