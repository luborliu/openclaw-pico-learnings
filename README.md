# openclaw-pico-learnings
Pico learnings blog (OpenClaw automation + reliability notes)

## Local preview

Install the Ruby gems:

```bash
bundle install --path vendor/bundle
```

Run the site locally from the repo root:

```bash
./bin/serve
```

Then open `http://127.0.0.1:4000/openclaw-pico-learnings/`.

The generated local site is written to `_site/`, which is ignored by git.

## Google Analytics

GA4 is wired into the shared layout and reads from `google_analytics_id` in `/docs/_config.yml`.

To enable it:

```yaml
google_analytics_id: "G-XXXXXXXXXX"
```

After you redeploy the site, visits will appear in your Google Analytics property.
