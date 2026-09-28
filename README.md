# Locate fibre blowing alarms along the duct run

A harbor-format benchmark task, authored for the AfterQuery Experts programme as data
annotation for AI evaluation. Task slug `afterquery/locate-blowing-alarms` — **Operations / Telecom Construction**.

An AI agent gets a container, this brief and nothing else. It works alone for
4 hours. Then a sealed verifier it never sees scores the result `0` or `1`.

## What it asks for

Where along the duct did each blowing alarm happen?

The full brief is in [`instruction.md`](instruction.md).

| | |
| --- | --- |
| **Deliverable** | `/app/dig_sheet.csv` |
| **Agent budget** | 4 hours (14400s) |
| **Expert estimate** | 5 hours |
| **Machine** | 1 CPU, 2048 MB RAM, 0 GPU |
| **Verifier budget** | 300s |
| **Tags** | `fibre-installation`, `machine-logs`, `record-forensics`, `data-cleaning`, `python` |

## Why it is hard

Turning a blowing machine's alarms into places on a duct route is real and ordinary work. Fibre installation engineers and the planners who raise the follow-up jobs do it after every difficult blow, and they do it from exactly these leavings: the route's duct schedule, the machine's log, the alarm stamps and the contractor's blowing procedure. The awkward parts of it here - a machine that counts feet, one that counts centimetres, one that writes what was paid out in each second and not the running total, one that writes seconds since the blow began and not the time of day - are the ones a hired-in fleet of machines actually has in it rather than invented ones.

<details>
<summary>Full note</summary>

The data is synthetic and was built truth first. 49 lengths of duct were scheduled on 5 routes and every alarm given the place where the cable head stood when it sounded, before any blow was logged; the blows were then logged, 18139 lines of them, the way five crews would have done it. Every answer is therefore exact by construction rather than a judgement made later, and the records are static files, so nothing is computed at marking time. Forty alarms are each to be given a distance from the A chamber and a kind of duct, right if the distance is within two metres and the duct is the scheduled one, and at most 4 may be wrong. No single distance is right for more than 2 of the forty. The arithmetic is not withheld. BLOWING.md gives all of it: how a route and its ducts are scheduled from the A chamber, how a blow is set up and logged, where the cable head is, where an alarm's place is, and the length of a foot. The work is in the doing. Taking the logs exactly as they stand gets 0 of 40. Five things about the logs have to be got from the logs themselves. None is written down anywhere, and each is settled by something in the records that cannot be right otherwise. The first is that one machine counts feet, so that it paid out three and a quarter times the length of its route. The second is that one counts centimetres, so that it paid out a hundred times it. The third is that one writes what was paid out in each second and not the running total, so that its figures go up and down and never pass two. The fourth is that one writes the seconds since the blow began and not the time of day. The fifth is not like them, and it is the one that decides the task. The procedure sets the machine up at the A chamber, the end a route's distances are reckoned from. 2 of the 5 routes were blown from the far chamber. Nothing in the records says which, and nothing breaks for a solver who takes the procedure at its word: the cable runs the length of its route either way, so every alarm still falls on the route and gets a distance and a duct. It is not tied to the four habits either - one of the two has a habit and the other is the log of the crew that otherwise kept to the procedure. No speed settles it: the machine holds its set speed in any duct, the average speed in one kind of duct is the average in another to within 4 in a hundred on every route from either end, and the plain spread of the speed is mostly the crew's own - it wanders over minutes, and now and then there is half a minute of fuss with the air or the pusher wherever the head happens to be - so a table of mean and spread by kind of duct says nothing either way. What settles it is how the cable runs. It goes sweetly into smoothwall duct and raggedly into old earthenware: the running total is right in both, but the length paid out from one second to the next is steady in the one and ragged in the other. Take how far each second on the move strays from the seconds either side of it, and take the middle value of that in each kind of duct: read from the right end the cable runs at least 3.0 times as raggedly in earthenware as in smoothwall on every route, and read from the wrong end the two come out about alike, lower by at least 2.0. It is in no column and in no mean, and the five routes are all different, so no blow can be laid against another. Blowing every route from its A chamber costs 16 of the 40 alarms, and blowing every route from the far end costs 24. tests/gates.py scores all 48 ways of reading those five. Every figure quoted here comes from its output. The whole account gets all 40. The best account that is not the whole one gets 32, so missing any single one of them fails. At most 4 may be wrong, which leaves room for a slip and still sits 4 clear of every partial account. It also refuses to pass unless the cable really does settle which end every route was blown from, since a reading the records did not settle would be a guess and not a finding.

</details>

## How it is verified

The deliverable is a data file. The verifier reads /app/dig_sheet.csv as text with the standard csv module and never executes, imports or loads anything the submission wrote, so there is nothing to sandbox and no way for a submission to influence its own score. 40 answers are marked, one for each alarm in follow_up.txt, each right if its distance is within two metres of the sealed one and its duct is the same.

<details>
<summary>Full note</summary>

The two metres are rounding room and nothing more: every one of the forty was chosen so that any account of the logs that gets something wrong puts it more than thirty metres out, and so that it stands ten metres clear of any change of duct read from either end. At most 4 may be wrong. The sealed answers live only in the verifier image at /tests/truth.json; the agent's container carries the procedure, the route list, the duct schedule, the logs, the alarms and the follow-up list, and nothing else. /tests is mode 700 in the verifier image, set at build time so it holds under a read-only root, and the agent's container has no mount of it at all; the verifier's log directory is likewise unreachable from the agent's side. There is nothing in the agent's container against which a solver could check an answer. No alarm's place is attested anywhere, no total has to come out and nothing has to balance, so a wrong set of places is exactly as self-consistent as the right one. They have to be worked out; they cannot be searched for. tests/gates.py is shipped and can be re-run. It scores all 48 readings of the records against the sealed answers and refuses to pass if the whole account does not get every one, if any other account comes within the allowance, or if guessing does. It reports the whole account at 40, the best account that is not it at 32, and the logs taken as they stand at 0. The records are static files from a seeded generator, so nothing is computed at marking time and the answer cannot drift between runs. The reference was run against the shipped bundle in the task's own container and got 40 of 40; an empty container scores 0.

</details>

<details>
<summary>Reference approach (spoiler)</summary>

solution/place_alarms.py reads only what the agent's container carries and writes /app/dig_sheet.csv. With solution/blowing.py and solution/ref.py it is under two hundred and fifty lines and imports nothing outside the standard library. It reads the duct schedule and the five logs and finds each log's habit from the log rather than assuming it: the log whose figures fall as well as rise, the two whose last figure matches the length of the route only in feet or only in centimetres, and the log whose times have no colons in them. It puts each right. For every blow it then takes how far the length paid out in each second on the move strays from the seconds either side of it, groups that by the kind of duct the cable head was in, and sets the middle value in earthenware over the middle value in smoothwall - once reading the counter from the A chamber and once from the far end - and keeps the end that gives the greater. It finds where the counter stood when each alarm sounded, turns that into a distance from the A chamber, and reads the duct off the schedule. Distances are carried as whole millimetres, so nothing turns on floating point. Run against the shipped records it gets 40 of 40.

</details>

## Layout

```
instruction.md          the brief - the only thing the agent is told
task.toml               artifacts, timeouts, resources, metadata
environment/            Dockerfile and data for the agent's container
solution/solve.sh       reference solution; scores 1 by construction
tests/                  verifier image, grading logic and ground truth
```

## Running it

With [harbor](https://github.com/laude-institute/harbor) installed and Docker running:

```bash
uv tool install harbor

harbor run -p . -a oracle -e docker   # reference solution - must score 1
harbor run -p . -a nop    -e docker   # nothing happens    - must score 0
```

Those two runs are the calibration the bundle has to satisfy: the oracle run substitutes
`solution/solve.sh` for the agent and must score `1`, the nop run does nothing and must
score `0`. Anything in between means the verifier is measuring the wrong thing.

## Reading this

`tests/` and `solution/` hold the answers. To try the task honestly, read only
`instruction.md` and `environment/`.

---

Part of a set of 38 tasks indexed at
[afterquery-data-annotation](https://github.com/helexdev1004/afterquery-data-annotation).
