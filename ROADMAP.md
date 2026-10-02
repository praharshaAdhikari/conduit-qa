# Roadmap: building the QA pipeline

The apps run and nothing checks them yet. Each step below adds one layer. Do them in order: each one
depends on the ones before it. A step is finished when its "done when" line is true.

Write down every decision and its reason as you go (in `docs/` in this repo). The reasons are the part
that carries over to the next project; the tools change.

## 1. Risk analysis and test strategy

Before any code. Use the app by hand, read the API reference, and write one page:

- what the system does and who uses it
- what could go wrong, and how bad each failure would be
- which risks get tested first, and which are deliberately left untested
- which kind of test covers each risk, and why that kind

**Done when:** someone who has never seen the system can read the page and tell what will be tested
first and why.

## 2. Unit tests in the API

Tests for logic that needs no database or network. Start with the risks ranked highest in step 1.

**Done when:** one command in `conduit-api` runs them in seconds, and each test fails when you break the
line it is about.

## 3. Integration tests against a real database

Tests for what only the database can prove: queries, filters, unique keys, cascading deletes,
transactions, and the migrations themselves.

**Done when:** one command starts a throwaway MySQL, builds the schema from `migrations/`, runs the
tests and removes the database, with nothing to set up by hand.

## 4. API end-to-end tests

Tests that send HTTP requests to the whole running API: status codes, error bodies, who is allowed to do
what.

**Done when:** every endpoint has its main success case and its main refusals (no token, someone else's
record, invalid input) covered.

## 5. CI in each app repo

Every pull request runs lint, typecheck, build and the tests from steps 2 to 4.

**Done when:** a pull request with a failing test cannot be merged.

## 6. Browser end-to-end tests

In this repo: tests that drive the web app against the real API and database, for the journeys that
matter most to a user.

**Done when:** one command runs them against the stack from `compose.yml` and produces a report with a
screenshot and trace for each failure.

## 7. Cross-repo gate

A pull request in either app repo builds the whole system (that repo at the pull request's commit, the
other at `dev`) and runs the browser tests from this repo.

**Done when:** a change in `conduit-api` that breaks the web app turns the pull request red before it is
merged.

## 8. Nightly run and known-bug policy

Decide what runs on every pull request and what runs nightly, and how a test for a known, unfixed bug is
kept without blocking unrelated work.

**Done when:** a normal run is green, any red means something changed, and the policy is written down.

## 9. Coverage of changed lines

New and edited lines must be exercised by tests; existing untested code does not fail the check.

**Done when:** a pull request that adds untested code is flagged, with the uncovered lines listed.

## 10. Breadth

One short exercise each, to learn what the pipeline above does not cover:

- contract testing between the web app and the API
- a performance test of one endpoint
- an accessibility pass on the main pages
- a security review of authentication and authorization
- an exploratory testing session with written notes and bug reports

**Done when:** each has a short write-up of what was found and whether it belongs in the pipeline.

## 11. Do it again on a different stack

Apply steps 1 to 6 to a project built with different tools.

**Done when:** you have a list of what had to change. Whatever did not change is the methodology;
whatever did was specific to this stack.
