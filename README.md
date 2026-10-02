# conduit-qa

The QA home of Conduit, a small blogging platform. It started as the
[RealWorld](https://github.com/realworld-apps/realworld) spec (users, articles, comments, tags, follows,
favorites) and adds roles and moderation on top. The system is three repos, checked out side by side:

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
docker compose up --build -d --wait           # MySQL, the API and the web app
docker compose exec api npm run seed:built    # optional: demo users and articles
```

The seed can be run again at any time; it adds only what is missing.

| What | Where |
| --- | --- |
| Web app | http://localhost:4100 |
| Moderation pages (moderators and admins) | http://localhost:4100/admin |
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

What each role may do, and what happens to suspended users and hidden articles, is written down in the
`conduit-api` README under "Roles and moderation". That section is the specification to test against.

```sh
docker compose logs -f api    # follow the API's log
docker compose down           # stop, keep the database
docker compose down -v        # stop and delete the database
```

The first start builds both apps and takes a few minutes. After changing code in `conduit-api` or
`conduit-web`, run the `up --build` command again.

To work on one app with live reload instead, follow that repo's README. It uses the same ports, so stop
this stack first.
