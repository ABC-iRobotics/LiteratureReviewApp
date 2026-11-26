# Review Helper — Literature Review Tools

Python-based tools for systematic and scoping literature reviews involving multiple bibliographic databases.

Antal Bejczy Center for Intelligent Robotics, Óbuda University, Budapest, Hungary.

## What this is

This repository contains two complementary Jupyter notebooks covering different stages of a literature review:

- **[arXiv API](arXiv_API.ipynb)** — a configurable search tool for retrieving bibliographic records directly from arXiv and exporting them in Atom XML format.
- **[Review Helper](Review_Helper.ipynb)** — an interactive application for importing and harmonizing bibliographic records, identifying duplicates, screening publications, comparing reviewer decisions, resolving disagreements, and extracting data.

The notebooks are intended for reuse across research domains. Search terms, eligibility criteria, full-text exclusion reasons, and data extraction fields can be defined by the user.

The workflow follows a **human-in-the-loop approach**: bibliographic processing and duplicate detection are supported by the software, while eligibility decisions, disagreement resolution, and data extraction remain under reviewer control. Neither notebook requires a large language model (LLM) or an API key.

## Repository structure

```text
arXiv_API.ipynb       Configurable arXiv search and XML export
Review_Helper.ipynb   Interactive literature review workflow
README.md             Documentation and usage instructions
```

## Dependencies

Both notebooks can be opened in [Google Colab](https://colab.research.google.com/) or run locally in a compatible Jupyter environment.

| Notebook | Requirements |
| --- | --- |
| `arXiv_API.ipynb` | Python standard library; internet connection |
| `Review_Helper.ipynb` | Python 3.12 or later; `pandas`, `numpy`, `openpyxl`, `ipywidgets` 7.x, `ipython` |

For local execution, install the notebook environment and the required packages:

```bash
pip install notebook pandas numpy openpyxl "ipywidgets>=7,<8" ipython
```

The current Review Helper implementation uses the `ipywidgets` 7.x file-upload interface. Version 8.x is not directly compatible with the upload code. If running in Google Colab, check that the runtime and installed widget version meet these requirements.

## Getting started

1. Open the desired `.ipynb` file in Google Colab or Jupyter.
2. If arXiv is one of your sources, configure and run `arXiv_API.ipynb`.
3. Export bibliographic records from any other databases used in the review.
4. Open `Review_Helper.ipynb` and run the code cell to launch the graphical interface.
5. Import the database exports through the **Database Uploader** and follow the review workflow described below.

The notebooks can also be used independently. The arXiv notebook is optional if arXiv is not part of your search strategy or if you already have a compatible export.

## 1. arXiv API

[`arXiv_API.ipynb`](arXiv_API.ipynb) provides a configurable interface to the public arXiv API. It constructs a Boolean search query from user-defined keyword groups, sends the request, and saves the original Atom XML response.

### Configuration

All user-adjustable parameters are located in the `CONFIGURATION` section of the notebook.

| Parameter | Description | Default |
| --- | --- | --- |
| `output_filename` | Name and location of the exported XML file | `arxiv_output.xml` |
| `search_fields` | Bibliographic fields to search | `["abs"]` |
| `term_groups` | Groups of related search terms | Empty; must be configured |
| `group_operator` | Boolean operator between keyword groups | `"AND"` |
| `start_index` | Index of the first result to retrieve | `0` |
| `max_results` | Maximum number of records requested per call | `2000` |
| `sort_by` | Result sorting criterion | `"submittedDate"` |
| `sort_order` | Ascending or descending order | `"descending"` |
| `request_timeout` | HTTP request timeout in seconds | `60` |
| `user_agent` | Identifier sent with the API request | `"ArxivLiteratureSearch"` |

Supported search fields include `ti` (title), `au` (author), `abs` (abstract), `co` (comment), `jr` (journal reference), `cat` (subject category), `rn` (report number), and `all` (all searchable fields).

### Example search

The notebook is distributed without predefined research keywords. Replace the empty `term_groups` list with terms relevant to your review.

```python
search_fields = ["abs"]

term_groups = [
    ["keyword1", "keyword2"],
    ["keyword3", "keyword4"]
]

group_operator = "AND"
```

Terms within each group are combined using `OR`, and the groups are combined using `group_operator`. Each term is searched in every selected field, with field-specific expressions also connected using `OR`.


### Running the search

Run the code cell after configuring the search. The notebook prints the generated search query and API URL, submits the request, and writes the response to `arxiv_output.xml` (or the configured filename).

The XML file can be imported directly into Review Helper without conversion.

For further details on query syntax and API usage, see the [arXiv API User's Manual](https://info.arxiv.org/help/api/user-manual.html).

## 2. Review Helper

[`Review_Helper.ipynb`](Review_Helper.ipynb) provides a graphical interface built with Jupyter widgets. It organizes the literature review into four modules.

### Supported databases

| Database | Input format | File extension |
| --- | --- | --- |
| PubMed | PubMed tagged plain text | `.txt` |
| arXiv | Atom XML | `.xml` |
| IEEE Xplore | RIS | `.ris` |
| Scopus | RIS | `.ris` |

The arXiv API notebook generates the Atom XML format expected by Review Helper. Other database exports must be obtained separately.

### Application workflow

#### 1. Database Uploader

Upload bibliographic exports, standardize their metadata, inspect individual database records, and merge the records into a common table. The application assigns record IDs and identifies duplicates using two methods:

- **Exact DOI matching** — flags subsequent records with the same non-empty DOI as confirmed duplicates.
- **Normalized title matching** — flags subsequent records with matching titles as suspected duplicates for manual verification.

Duplicate classifications and the corresponding record IDs are retained in the workbook for traceability.

#### 2. Screening

Upload the Excel workbook generated by the Database Uploader and perform title-and-abstract screening followed by full-text screening.

Reviewers can record inclusion and exclusion decisions, identify additional duplicates manually, and define full-text exclusion reasons. The screening interface provides summary counts that can support PRISMA reporting.

#### 3. Compare Decisions (optional)

Import independently screened Excel workbooks, identify disagreements in title-and-abstract and full-text decisions, and resolve them by selecting the appropriate reviewer decision.

For multi-reviewer reviews, each reviewer should start with an independent copy of the same initial workbook. Record IDs and row order must be preserved across all copies. Use the resolved workbook for subsequent data extraction.

#### 4. Data Extraction

Define the variables to be collected from the included publications. The application generates corresponding input fields and allows reviewers to record and update extracted information for articles included at both screening stages.

The extraction fields are user-defined rather than tied to a specific research question or domain.

## Data management

Review data are stored in Excel (`.xlsx`) workbooks so that the review can be inspected, resumed, and archived between stages.

| File or worksheet | Contents |
| --- | --- |
| `arxiv_output.xml` | Raw arXiv API response |
| `ReviewOutput.xlsx` | Initial bibliographic records and duplicate classifications; updated during subsequent stages if the same workbook is used |
| `Resolved_Review.xlsx` | Consolidated screening decisions after reviewer disagreement resolution |
| `Final` worksheet | Harmonized metadata, duplicate classifications, screening decisions, exclusion reasons, and extracted variables |

Use the application's **Save Final** function before leaving a stage. In Google Colab, download the generated files from the Files panel before ending the runtime, as runtime storage is temporary.


## Scope and limitations

- The arXiv notebook retrieves bibliographic metadata rather than full-text PDFs. Search completeness depends on the query, API response, and retrieval settings.
- Review Helper supports the four database export formats listed above. The RIS parser distinguishes Scopus by its `DB` tag and otherwise treats the file as IEEE Xplore; other RIS sources are not explicitly supported.
- Duplicate detection uses exact DOI and normalized title matching, not fuzzy matching or semantic similarity. Variant titles, missing DOIs, and inconsistent DOI formatting may require manual review.
- Screening and extraction are reviewer-driven. Full texts must be obtained and assessed separately.
- Screening statistics support, but do not replace, the verification of PRISMA flow counts and methodological reporting. Reviewers remain responsible for the completeness and accuracy of the final review.

For reproducibility, retain the original search queries, search dates, database exports, review criteria, and intermediate workbooks.

## Citation

If you use these tools in your research, please cite the associated manuscript:

> E. Lukács, Z. Zrubka, and T. Haidegger, “Applicability of a Literature Review Support Software Tool for Multi-Database Literature Screening and Decision Tracking,” manuscript in preparation for *Acta Polytechnica Hungarica*.

## Funding

The project was partially supported by 2024-1.2.8-TÉT-IPARI-CN-2025-00040, implemented with the support of the Hungarian Ministry of Culture and Innovation from the National Research, Development and Innovation Fund.

Project 2024-1.2.3-HU-RIZONT-00069 has been implemented with the support provided by the Ministry of Culture and Innovation of Hungary from the National Research, Development and Innovation Fund, financed under the 2024-1.2.3-HU-RIZONT funding scheme.

Project No. 2026-2.1.1-EKÖP-2026-00012-D-139 has been implemented with the support provided by the Ministry of Science and Technology of Hungary from the National Research, Development and Innovation Fund, financed under the 2026-2.1.1-EKÖP-2026-00012 University Research Scholarship Programme (EKÖP) funding scheme.
