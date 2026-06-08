# JOSE submission — OpenChapters

Working folder for the *Journal of Open Source Education* submission
describing the OpenChapters web platform.

## Files

- `paper.md` — the manuscript, in JOSE's required pandoc-Markdown format with
  YAML frontmatter.
- `paper.bib` — BibTeX references cited from `paper.md` via pandoc-style
  `[@key]` citations.

## Before submitting

1. **Fill in author metadata** in the YAML frontmatter of `paper.md`:
   - Replace the placeholder `0000-0000-0000-0000` ORCID with your real ORCID.
   - Add co-authors (with their ORCIDs and affiliations) if any contributed
     substantively to the platform.
2. **Review the acknowledgements** — funding sources (ONR Vannevar Bush
   Fellowship, NSF DMR-1904629, NSF DMR-2203378, Bertucci Professorship)
   are pre-filled from the project README. Add any student developers or
   contributing chapter authors who deserve named credit.
3. **Verify references** in `paper.bib` — especially the textbook-cost
   citation, which currently points at a 2014 PIRG report; a more recent
   Florida Virtual Campus or BCcampus survey would strengthen the statement
   of need.
4. **Update the date** at the top of `paper.md` to the actual submission date.
5. **Tag the OpenChapters repo** at the version being submitted, and note
   that release/tag in the submission form.

## How to render locally (optional)

JOSE compiles the paper for you during review, but if you want to preview:

```bash
docker run --rm \
  -v "$PWD":/data \
  -u $(id -u):$(id -g) \
  -e JOURNAL=jose \
  openjournals/inara \
  -o pdf,crossref paper.md
```

`openjournals/inara` is the official JOSE/JOSS build container; it produces
the same PDF the editors will see. The `-e JOURNAL=jose` flag is required:
the container **defaults to `joss`**, so without it the rendered PDF is
branded as the *Journal of Open Source Software* instead of the *Journal of
Open Source Education*.

## How to submit

1. Place `paper.md` and `paper.bib` at the **root** of the OpenChapters
   GitHub repository (or in a `paper/` subdirectory) and push.
2. The software repository is licensed under **BSD 3-Clause** (an
   OSI-approved license), which satisfies JOSE's licensing requirement. The
   *chapter content* in the separate `OpenChapters/OpenChapters` monorepo
   remains under CC BY-NC-SA 4.0.
3. Submit at <https://jose.theoj.org/papers/new> with the repository URL and
   the version tag.
4. A JOSE editor will assign reviewers; review happens in a public GitHub
   issue on `openjournals/jose-reviews`.

## Useful links

- JOSE submission guidelines:
  <https://jose.theoj.org/about#author_guidelines>
- JOSE paper template (canonical reference):
  <https://github.com/openjournals/jose-papers>
- Example accepted JOSE papers:
  <https://jose.theoj.org/papers>
