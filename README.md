# AI Resume Screener

An LLM-powered tool that reads resumes (PDF or DOCX), parses them into structured data, and scores each candidate against a job description. Candidates are ranked, and the top and bottom matches are printed at the end.

Built with Python, the [Groq](https://groq.com/) API, and Pydantic.

## Features

- **Job description analysis**: turns a free-text job description into structured data (role, required and preferred skills, minimum experience, education, responsibilities).
- **Resume parsing**: extracts name, contact details, skills, experience (including internships), education, projects and certifications. Parsing is based on meaning, not exact section headings, so resumes with different layouts work.
- **Candidate scoring**: compares each parsed resume with the job and returns a match score from 0 to 100 plus details (matching skills, missing skills, whether the experience requirement is met, and a short verdict).
- **Ranking**: sorts all candidates by score and prints the top 2 and lowest 2.
- **Multiple formats**: reads `.pdf` (via `pypdf`) and `.docx` (via `python-docx`, including text inside tables).
- **Validated output**: LLM responses are requested as JSON and validated with Pydantic models.

## How it works

```
Job description ──► LLM ──► structured job (JobD)
                                   │
resume/*.pdf|docx ──► text ──► LLM ──► structured resume (Resume)
                                   │
                    structured job + structured resume ──► LLM ──► MatchResult (score + details)
                                   │
                           sort by score ──► print top 2 / lowest 2
```

For each resume the script makes two LLM calls (parse, then score), with a short `time.sleep` between calls to stay within API rate limits.

## Tech stack

| Purpose | Library |
| --- | --- |
| LLM API | [`groq`](https://pypi.org/project/groq/) (model: `openai/gpt-oss-120b`) |
| Data validation | [`pydantic`](https://pypi.org/project/pydantic/) |
| PDF reading | [`pypdf`](https://pypi.org/project/pypdf/) |
| DOCX reading | [`python-docx`](https://pypi.org/project/python-docx/) |
| Env variables | [`python-dotenv`](https://pypi.org/project/python-dotenv/) |

## Getting started

### Prerequisites

- Python 3.11 or newer
- A Groq API key (free to create at [console.groq.com](https://console.groq.com/))

### Installation

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

Using [uv](https://docs.astral.sh/uv/) (recommended, since the project includes a `uv.lock`):

```bash
uv sync
```

Or with pip:

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install groq pydantic pypdf python-docx python-dotenv
```

### Configuration

Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_api_key_here
```

> Never commit your `.env` file. Make sure it is listed in `.gitignore`.

### Usage

1. Put the resumes you want to screen (`.pdf` or `.docx`) in the `resume/` folder. Other file types are skipped.
2. Edit the `job_description` variable in `resume_screener.py` to the role you are hiring for.
3. Run the screener from the project root (the `resume/` folder is resolved relative to where you run it):

```bash
uv run resume_screener.py
# or, with pip: python resume_screener.py
```

The script prints each file as it is processed, followed by the ranking:

```
Processing: <resume file name>
Score: <0-100>
...
TOP 2 CANDIDATES
<name> - <score> %
<details>
LOWEST 2 CANDIDATES
<name> - <score> %
<details>
```

## Project structure

```
.
├── resume_screener.py   # Main script: JD analysis, resume parsing, scoring, ranking
├── main.py              # Placeholder entry point (not used by the screener)
├── resume/              # Drop resumes here (.pdf / .docx)
├── pyproject.toml       # Project metadata and dependencies
├── uv.lock              # Locked dependency versions
├── .python-version      # Python version used for development
└── README.md
```

## Customization

- **Different job**: replace the `job_description` string in `resume_screener.py`.
- **Different model**: change the `model` variable (any Groq-hosted model that supports JSON output).
- **Rate limits**: adjust the `time.sleep(5)` calls if you hit API limits or want to run faster.
- **Scoring criteria**: edit the prompt in `final_score()` to weight skills, experience or education differently.

## Limitations

- Scores come from an LLM, so they can vary between runs and should support, not replace, human review.
- Scanned or image-only PDFs have no extractable text and are not supported (there is no OCR step).
- Resumes are processed one at a time, which is slow for large batches.
- The job description is hardcoded in the script rather than passed in as an input.

## Privacy note

Resume contents are sent to the Groq API for processing. Only screen resumes you have permission to process, and do not commit real candidates' resumes to a public repository.

## Contributing

Issues and pull requests are welcome.