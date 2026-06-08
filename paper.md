---
title: 'OpenChapters: A web platform for on-demand, open-source PDF/HTML chapter collections built from LaTeX source'
tags:
  - open educational resources
  - OER
  - open textbooks
  - LaTeX
  - arara
  - materials science
  - engineering education
  - Django
  - React
authors:
  - given-names: Marc
    non-dropping-particle: De
    surname: Graef
    orcid: 0000-0002-4721-6226
    corresponding: true
    affiliation: 1
affiliations:
  - name: Department of Materials Science and Engineering, Carnegie Mellon University, Pittsburgh, PA, USA
    index: 1
date: 13 May 2026
bibliography: paper.bib
---

# Summary

`OpenChapters` is a free, open-source web platform that lets instructors and
self-learners assemble customized PDF and/or HTML chapter collections on demand from a shared library
of peer-contributed LaTeX chapters. Browsing a graphical chapter catalog,
users select chapters of interest, organize them into parts, supply their own
book and part titles as well as (worked) examples, 
and submit the result for typesetting. A server-side
build worker clones the requested chapter sources from a GitHub monorepo,
invokes the `arara` build pipeline [@cereda:arara] against a Jinja-generated
`main.tex`, and emails the user a link to the finished PDF file or HTML zip file, which is also
stored in their personal library on the platform. Chapters are released under
the Creative Commons CC BY-NC-SA 4.0 license, so the resulting books inherit
an open, share-alike status. The platform software itself is released under
the BSD 3-Clause license. There is no payment system and no commercial
gating; the platform is entirely free to use.

The initial chapter library is focused on undergraduate and graduate
materials science and engineering, but the platform itself is
domain-agnostic: any community willing to maintain a LaTeX chapter monorepo
and a small metadata file per chapter can deploy the system for their own
discipline.

# Statement of need

Commercial textbooks in technical disciplines routinely cost students US$100
to US$300 [@senack:cost-of-textbooks], and rarely match any one instructor's
syllabus on the first try. Faculty respond by assigning subsets of chapters,
mixing readings from multiple sources, or distributing photocopied notes—all
workarounds that fragment the student experience and offer no consistent
typography, indexing, or cross-references.

Existing open educational resource (OER) platforms have done a great deal to
reduce textbook cost. OpenStax [@openstax], LibreTexts [@libretexts], and the
Open Textbook Library [@otl] curate full open textbooks, and authoring
toolchains such as PreTeXt [@pretext] produce print-quality output from a
single source. These platforms address the **cost** problem effectively, but
they generally deliver a fixed-scope textbook: the instructor adopts the
book whole, or not at all.

`OpenChapters` is designed for a different mode of use: **chapter-level
composition**. The unit of curation is the chapter, not the book. Instructors
select only the chapters they teach, in the order they teach them, with their
own book title and part structure, and obtain a single professionally typeset
PDF or HTML site that students treat as their course textbook. Because the build is
LaTeX-native—and uses the same `arara` directives, packages, and bibliography
machinery that the original chapter authors used—equations, figures,
cross-references, citations, and the index are preserved correctly across
arbitrary chapter combinations. To the best of our knowledge, no other open
educational platform combines (i) a chapter-level browse-and-assemble
interface, (ii) a true LaTeX-native typesetting backend that handles
cross-references and bibliographies across user-chosen chapter sets,
including automatic resolution of prerequisite chapters, and
(iii) a fully open contribution model with chapters hosted as a public
monorepo.

The combination is intended to serve three audiences:

- **Instructors** who want a personalized textbook for their specific course
  without committing to a single publisher's table of contents.
- **Students** who pay nothing and receive a coherent, professionally
  typeset PDF or HTML site rather than a stack of photocopies.
- **Authors** who contribute chapters to the shared monorepo and receive
  a DOI for each chapter they contribute.

# Functionality

The platform is structured around four user-facing surfaces and one backend
build pipeline.

**Chapter browser.** A graphical catalog of available chapters, each with a
hover-to-preview table of contents pulled from the chapter's compiled
metadata. The catalog is synced nightly from `chapter.json` files in the
OpenChapters GitHub monorepo, so new contributions appear automatically.

**Book editor.** Authenticated users build a book by adding chapters to a
shopping-cart-style selection, organizing them into named parts via
drag-and-drop, and supplying their own book title, subtitle, and part
titles. A *structure-preview* build option produces a PDF containing only
the front matter, part titles, chapter headings, and table of contents in
under a minute, giving the user a fast read of the book's macro structure
before committing to a full build. Chapters declare two kinds of
inter-chapter relationships in their metadata: *foundational
prerequisites*, which the platform resolves transitively and includes
automatically in a dedicated "Foundations" part (topologically ordered, so
each prerequisite precedes the chapter that needs it), and *topical
cross-references*, which are surfaced to the user as optional suggestions
but never force-included. This keeps cross-references and citations
resolvable across arbitrary chapter selections without obliging the user to
hunt down prerequisites manually.

**Build pipeline.** When the user submits the book, a Celery worker [@celery]
running inside a container with TeX Live and `arara` installed clones the
requested chapter sources from GitHub, renders a `main.tex` from a Jinja2
template containing the user's selections, and runs the `arara` directive
sequence (`pdflatex` → `biber` → `makeindex` → `pdflatex`). The arara
pipeline gives chapter authors fine-grained control over build steps from
inside the chapter source itself, which avoids hard-coding build logic in
the platform. The same chapter sources can also be typeset as a browsable
HTML site through a parallel `lwarp`-based pipeline [@dunn:lwarp]: equations
are rendered with MathJax and figures emitted as SVG, so the HTML preserves
the same cross-references, citations, and index as the PDF. On success, the
resulting PDF and/or HTML output is uploaded to object storage—the HTML as a
downloadable zip that the user can also read online on the platform—the user
is emailed a link, and the build is recorded in their personal library.

**Personal library.** Users can re-download past builds, fork a previous
book as the starting point for a new one, and view the version of each
chapter that was used in a given build—important because chapters evolve
and a course needs to be reproducible across semesters. To make that
reproducibility durable, a user can *freeze* a completed build: the platform
copies its artifacts, together with a per-chapter record of the exact source
commit used, into an immutable snapshot served from a stable, login-free
share URL. An instructor can distribute that link to students or embed it in
a learning-management system, and it continues to resolve to the identical
book even as the underlying chapters—or the user's own editable copy—change
or are deleted.

**Open API.** All user-facing operations are also available through a
documented REST API with a self-hosted Swagger UI and an OpenAPI 3 schema,
enabling external tools (course management systems, syllabus generators) to
assemble books programmatically.

The codebase is implemented in a Django and Django REST Framework on the
backend, React and Vite on the frontend, with PostgreSQL for data,
RabbitMQ as the Celery broker, and dedicated worker containers for the
LaTeX build. The entire stack is reproducible via Docker Compose, and a
parallel staging stack mirrors the production deployment for safe
verification of upgrades. Deployment guides, an administrator guide, a user
guide, and an API reference are maintained alongside the code.

# Acknowledgements

The OpenChapters project began with partial financial support from a
Vannevar Bush Faculty Fellowship (ONR award N00014-16-1-2821), continued
with partial funding from National Science Foundation award DMR-1904629,
and is currently supported by National Science Foundation award
DMR-2203378. MDG gratefully acknowledges support from the John and Claire
Bertucci Distinguished Professorship in Engineering at Carnegie Mellon
University. We thank current/future authors who have contributed/will contribute chapters to the
OpenChapters monorepo, and the maintainers of the `arara`, `biber`, and
TeX Live projects on which the build pipeline depends.

# References
