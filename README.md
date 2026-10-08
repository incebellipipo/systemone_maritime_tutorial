# System One for Marine Autonomy

A short tutorial on **System One models**: AI decision models that take content plus typed questions and return typed answers with probabilities, not generated text. The case study is collision avoidance for an autonomous surface vessel.

It covers:

- how System One responses differ from LLM responses
- the shared `/v1/systemone` request format
- building a request step by step, from one yes/no question to a full encounter
- writing clearer requests
- gating and combining the answers, with the maths written out

The model is a fast decision component, not the navigator. This is decision support, not a certified collision-avoidance system.

Start from [docs/index.md](docs/index.md), or read it as a website (see below).

## Layout

```text
docs/        the tutorial (markdown; also the site's pages)
_layouts/    page layout
_includes/   MathJax and Mermaid
assets/      stylesheet (LaTeX-article look) and Computer Modern fonts
llms.txt     index for language models
src/         reserved (empty)
notebooks/   reserved (empty)
```

## Website

The site is built with Jekyll. It has a LaTeX-article look, after [Gradfolio](https://github.com/jitinnair1/gradfolio) and [LatexJekyll](https://github.com/Hammie217/LatexJekyll), with a light/dark switch.

**Preview locally** with Docker:

```bash
docker compose up
# open http://localhost:4000/systemone_for_marine_autonomy/
```

Edits to `docs/` reload in the browser. Without Docker, run `bundle install` and `bundle exec jekyll serve`.

**Publish** with GitHub Pages: push to `main`, then set *Settings > Pages > Source* to *GitHub Actions*. The workflow in `.github/workflows/pages.yml` builds and deploys the site at `https://<user>.github.io/<repository>/`, or under your custom domain if your account has one. Forks work the same way.

To customise, edit `title`, `author` and the optional `home_url` in `_config.yml`.

## License

MIT. See [LICENSE.md](LICENSE.md).
