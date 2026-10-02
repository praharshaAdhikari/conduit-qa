# conduit-qa

The QA home of Conduit, a small blogging platform. It started as the
[RealWorld](https://github.com/realworld-apps/realworld) spec (users, articles, comments, tags, follows,
favorites) and adds roles and moderation, and paid memberships and tips through a payment provider, on
top. The system is three repos, checked out side by side:

| Repo | What it is |
| --- | --- |
| `conduit-api` | The API: NestJS, TypeORM, MySQL |
| `conduit-web` | The web app: React |
| `conduit-qa` (this one) | Starts the whole system; the cross-app tests and the pipeline are built here |

**There are no tests or automation yet, on purpose.** This is a practice system for learning how a QA
pipeline is planned and built, one layer at a time. [ROADMAP.md](ROADMAP.md) lists the layers in order.

## Start the system

Needs Docker, and the three repos next to each other.

```sh
docker compose up --build -d --wait           # MySQL, a mail inbox, the API and the web app
docker compose exec api npm run seed:built    # optional: demo users and articles
```

The seed can be run again at any time; it adds only what is missing.

| What | Where |
| --- | --- |
| Web app | http://localhost:4100 |
| Moderation pages (moderators and admins) | http://localhost:4100/admin |
| Membership page (when logged in) | http://localhost:4100/membership |
| Mail inbox: every email the API sends | http://localhost:4025 |
| API | http://localhost:4000/api |
| API reference (Swagger) | http://localhost:4000/api/docs |
| Health check | http://localhost:4000/api/health |
| MySQL | `127.0.0.1:4306`, database `conduit`, user `conduit`, password `conduit` |

Demo logins after seeding, all with the password `password123`:

| Email | Who |
| --- | --- |
| `alice@example.com`, `bob@example.com`, `carol@example.com` | Ordinary users with articles |
| `mod@example.com` | A moderator: can suspend users and hide articles |
| `admin@example.com` | An admin: a moderator who can also change roles |
| `dave@example.com` | A suspended user; his one article is hidden |
| `erin@example.com` | A paying member; she can read alice's members-only article |

The seed also leaves a paid tip to alice from a guest, `reader@example.com`, who has no account.

The rules of the system are written down in the `conduit-api` README: "Rules the RealWorld spec leaves
open", "Roles and moderation", "Memberships and payments" and "Tips". Those sections are the specification
to test against.

## Email

No email leaves this machine. The API sends everything to the mail inbox at http://localhost:4025, which
shows each message and also has an API (`curl -s localhost:4025/api/v1/messages`). That is where the code
for a guest's tip arrives, and the receipts.

## Payments

Nothing has to be set up. The API uses its built-in fake payment provider, so paying for a membership works
with no account and no network, and no money moves. The checkout page it shows has buttons to pay, to
decline the card and to cancel, and lets you choose how the webhook is delivered: straight away, late,
twice or never.

What a real provider does on its own schedule can be done on request, with no login:

```sh
curl -s localhost:4000/api/fake-pay/subscriptions                                # find a subscription's id
curl -s -X POST localhost:4000/api/fake-pay/subscriptions/<id>/renew            # the next period is paid
curl -s -X POST localhost:4000/api/fake-pay/subscriptions/<id>/fail             # the renewal payment fails
curl -s -X POST localhost:4000/api/fake-pay/subscriptions/<id>/end              # the subscription is over
curl -s localhost:4000/api/fake-pay/events                                      # every webhook it produced
curl -s -X POST localhost:4000/api/fake-pay/events/<id>/resend                  # deliver one again
```

A webhook that is never delivered leaves the API out of step with the provider. The reconcile job puts
that right, and also does what only the passing of time causes (ending an overdue membership, closing an
abandoned checkout). It runs every night at 03:00 UTC; to run it now:

```sh
docker compose exec api npm run -s membership:reconcile:built    # prints the run's report
```

An admin can also run it from the moderation pages (Memberships).

The full list, the delivery options and the webhook's signature are in the `conduit-api` README under "The
fake payment provider". To use Stripe test mode instead, see "Stripe test mode" there and set the same
three variables on the `api` service in `compose.yml`.

```sh
docker compose logs -f api    # follow the API's log
docker compose down           # stop, keep the database
docker compose down -v        # stop and delete the database
```

The first start builds both apps and takes a few minutes. After changing code in `conduit-api` or
`conduit-web`, run the `up --build` command again.

To work on one app with live reload instead, follow that repo's README. It uses the same ports, so stop
this stack first.
