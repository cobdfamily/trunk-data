# trunk-data

[![test](https://github.com/cobdfamily/trunk-data/actions/workflows/test.yml/badge.svg)](https://github.com/cobdfamily/trunk-data/actions/workflows/test.yml)

Per-team production data for `cobdfamily/trunk` — menus,
extensions, documents, audio prompts, and shared Jinja2
templates. The `trunk` deploy host clones this repo and
bind-mounts it as `/app/data` inside the trunk container.

```
layouts/                 layout templates wrapping every render
templates/               shared menu / extension / error templates
teams/<team>/
  team.yaml              optional, per-team settings (signature
                         verification, ...)
  menus/<name>.yaml      Twilio Gather config
  extensions/<n>/        per-extension profile + audio prompts
    profile.yaml
    audio/<file>
  documents/<name>.xml.j2
  audio/<file>
```

## End-to-end tests

`docker-compose.yaml` brings up `cobdfamily/trunk` with
this checkout mounted as its data tree, plus a
`cobdfamily/talkshow` alongside it for production-shape
parity. `tests/test_e2e.py` walks the menu / extension /
audio paths through trunk and asserts the rendered TwiML.

```sh
docker compose up -d

python3 -m venv tests/.venv
tests/.venv/bin/pip install -r tests/requirements.txt
tests/.venv/bin/python -m pytest tests/test_e2e.py -v

docker compose down -v
```

The suite locks the data tree against the regression that
bit production once: `{{ data.. }}` smudges in the shared
templates from a stale `trunk-migrate` run that broke
every menu and extension render.

## CI

`.github/workflows/test.yml` runs the E2E suite on push,
on PR, and nightly at 07:00 UTC. The nightly catches a
`trunk:latest` or `talkshow:latest` regression that
breaks rendering of this data tree within ~24h, instead
of waiting for the next push to surface it.

## License

AGPL-3.0 — see `LICENSE`.
