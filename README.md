# seta-careers-platform-repomx
This file is a merged representation of a subset of the codebase, containing specifically included files, combined into a single document by Repomix.
The content has been processed where security check has been disabled.

# File Summary

## Purpose
This file contains a packed representation of a subset of the repository's contents that is considered the most important context.
It is designed to be easily consumable by AI systems for analysis, code review,
or other automated processes.

## File Format
The content is organized as follows:
1. This summary section
2. Repository information
3. Directory structure
4. Repository files (if enabled)
5. Multiple file entries, each consisting of:
  a. A header with the file path (## File: path/to/file)
  b. The full contents of the file in a code block

## Usage Guidelines
- This file should be treated as read-only. Any changes should be made to the
  original repository files, not this packed version.
- When processing this file, use the file path to distinguish
  between different files in the repository.
- Be aware that this file may contain sensitive information. Handle it with
  the same level of security as you would the original repository.

## Notes
- Some files may have been excluded based on .gitignore rules and Repomix's configuration
- Binary files are not included in this packed representation. Please refer to the Repository Structure section for a complete list of file paths, including binary files
- Only files matching these patterns are included: index.html, docs/data-audit-report.md, reports/source-health-report.json, CareerHub SA AI Development Strategy.txt, data/clean/guides.json, data/guides.json, data/sources.json, data/opportunities.json, data/clean/opportunities.json, deploy.sh, README.md, crawler/output/rejected.json, docs/architecture/Repository-Workflow-Optimization-Framework.md, Repository Workflow Optimization Framework.txt, docs/source-registry.md, scripts/normalize_dataset.js, scripts/source-health-check.js, sitemap.xml, docs/architecture/frontend.md, scripts/validators/source-validator.js, scripts/validate-content.js, data/clean/provinces.json, data/provinces.json, docs/UPSCALE_WORKFLOW.md, sitemap-generator.js, scripts/check-links.js, crawler/index.js, docs/architecture/overview.md, docs/phases/backlog.md, docs/schema-draft.json, crawler/reporting/fetch-diagnostics.js, docs/source-governance.md, validate-content.js, docs/modules/module-index.md, scripts/source-audit.js, crawler/pipeline/staging.js, docs/architecture/data.md, crawler/pipeline/normalize.js, data/clean/categories.json, data/categories.json, 404.html, crawler/core/fetcher.js, crawler/core/source-loader.js, .gitignore, docs/templates/codex-task-template.md, docs/phases/dependency-map.md, crawler/core/router.js, crawler/parsers/html-parser.js, docs/blockers.md, crawler/parsers/pdf-parser.js, crawler/output/fetch-diagnostics.json, docs/rules/boundaries.md, docs/context/ui-context.md, docs/templates/rulesets.md, docs/context/scripts-context.md, package.json, docs/context/data-context.md, .github/workflows/quality.yml, crawler/pipeline/validate.js, docs/rules/data-access-rules.md, docs/rules/context-budget-rules.md, reports/source-audit-report.json, robots.txt, crawler/output/crawl-report.json, crawler/config/crawler-config.js, CNAME, crawler/output/staging.json
- Files matching patterns in .gitignore are excluded
- Files matching default ignore patterns are excluded
- Security check has been disabled - content may contain sensitive information
- Files are sorted by Git change count (files with more changes are at the bottom)

# Directory Structure
```
.github/
  workflows/
    quality.yml
crawler/
  config/
    crawler-config.js
  core/
    fetcher.js
    router.js
    source-loader.js
  output/
    crawl-report.json
    fetch-diagnostics.json
    rejected.json
    staging.json
  parsers/
    html-parser.js
    pdf-parser.js
  pipeline/
    normalize.js
    staging.js
    validate.js
  reporting/
    fetch-diagnostics.js
  index.js
data/
  clean/
    categories.json
    guides.json
    opportunities.json
    provinces.json
  categories.json
  guides.json
  opportunities.json
  provinces.json
  sources.json
docs/
  architecture/
    data.md
    frontend.md
    overview.md
    Repository-Workflow-Optimization-Framework.md
  context/
    data-context.md
    scripts-context.md
    ui-context.md
  modules/
    module-index.md
  phases/
    backlog.md
    dependency-map.md
  rules/
    boundaries.md
    context-budget-rules.md
    data-access-rules.md
  templates/
    codex-task-template.md
    rulesets.md
  blockers.md
  data-audit-report.md
  schema-draft.json
  source-governance.md
  source-registry.md
  UPSCALE_WORKFLOW.md
reports/
  source-audit-report.json
  source-health-report.json
scripts/
  validators/
    source-validator.js
  check-links.js
  normalize_dataset.js
  source-audit.js
  source-health-check.js
  validate-content.js
.gitignore
404.html
CareerHub SA AI Development Strategy.txt
CNAME
deploy.sh
index.html
package.json
README.md
Repository Workflow Optimization Framework.txt
robots.txt
sitemap-generator.js
sitemap.xml
validate-content.js
```

# Files

## File: .github/workflows/quality.yml
````yaml
name: Quality Gates

on:
  pull_request:
  push:
    branches:
      - main
      - work
  workflow_dispatch:

permissions:
  contents: read

jobs:
  content-and-sitemap:
    name: Validate content and sitemap
    runs-on: ubuntu-latest

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Validate JSON content
        run: npm run validate

      - name: Check opportunity links
        run: npm run links

      - name: Regenerate sitemap
        run: npm run sitemap

      - name: Ensure generated files are committed
        run: git diff --exit-code -- sitemap.xml
````

## File: crawler/config/crawler-config.js
````javascript
module.exports = Object.freeze({
  timeout: 15000,
  retries: 3,
  concurrency: 5,
  maxRedirects: 5,
});
````

## File: crawler/core/fetcher.js
````javascript
const axios = require('axios');

function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function fetchOnce(source, config) {
  const response = await axios.get(source.url, {
    timeout: config.timeout,
    maxRedirects: config.maxRedirects,
    responseType: 'arraybuffer',
    validateStatus: () => true,
    headers: {
      'User-Agent': 'SETA-Careers-Platform-Crawler/1.0',
      Accept: 'text/html,application/pdf;q=0.9,*/*;q=0.8',
    },
  });

  return {
    status: response.status,
    contentType: response.headers['content-type'] || '',
    body: Buffer.from(response.data),
    url: response.request && response.request.res && response.request.res.responseUrl ? response.request.res.responseUrl : source.url,
  };
}

async function fetchSource(source, config, log = () => {}, events = {}) {
  let lastError = null;

  for (let attempt = 1; attempt <= config.retries; attempt += 1) {
    try {
      const response = await fetchOnce(source, config);

      if (response.status >= 200 && response.status < 300) {
        log(events.FETCH_SUCCESS || 'FETCH_SUCCESS', { source_id: source.source_id, url: response.url, status: response.status, attempt });
        return { ok: true, ...response };
      }

      lastError = new Error(`HTTP ${response.status}`);
      log(events.FETCH_FAILURE || 'FETCH_FAILURE', { source_id: source.source_id, url: source.url, status: response.status, attempt, error: lastError.message });
    } catch (error) {
      lastError = error;
      log(events.FETCH_FAILURE || 'FETCH_FAILURE', { source_id: source.source_id, url: source.url, attempt, error: error.message });
    }

    if (attempt < config.retries) {
      await sleep(250 * attempt);
    }
  }

  return {
    ok: false,
    status: null,
    contentType: '',
    body: null,
    url: source.url,
    error: lastError ? lastError.message : 'Unknown fetch failure',
  };
}

module.exports = {
  fetchSource,
};
````

## File: crawler/core/router.js
````javascript
function detectContentType(fetchResult) {
  const contentType = (fetchResult.contentType || '').toLowerCase();
  const url = (fetchResult.url || '').toLowerCase();
  const body = fetchResult.body;

  if (contentType.includes('text/html') || contentType.includes('application/xhtml+xml')) return 'html';
  if (contentType.includes('application/pdf')) return 'pdf';
  if (url.endsWith('.pdf')) return 'pdf';

  if (Buffer.isBuffer(body)) {
    const signature = body.subarray(0, 5).toString('utf8');
    if (signature === '%PDF-') return 'pdf';

    const sample = body.subarray(0, 512).toString('utf8').toLowerCase();
    if (sample.includes('<!doctype html') || sample.includes('<html')) return 'html';
  }

  return 'unknown';
}

function routeContent(fetchResult, log = () => {}, events = {}) {
  const contentType = detectContentType(fetchResult);

  if (contentType === 'unknown') {
    log(events.UNSUPPORTED_CONTENT || 'UNSUPPORTED_CONTENT', {
      url: fetchResult.url,
      reason: 'UNSUPPORTED_CONTENT_TYPE',
      content_type: fetchResult.contentType || '',
    });
  }

  return contentType;
}

module.exports = {
  detectContentType,
  routeContent,
};
````

## File: crawler/core/source-loader.js
````javascript
const path = require('path');
const { loadSources, validateSource } = require('../../scripts/validators/source-validator');

const DEFAULT_SOURCE_PATH = path.resolve(__dirname, '..', '..', 'data', 'sources.json');
const INVALID_SOURCE_REGISTRY = 'INVALID_SOURCE_REGISTRY';

function toCrawlerSource(source) {
  return {
    source_id: source.source_id,
    name: source.name,
    url: source.url,
    type: source.type,
    crawl_strategy: source.crawl_strategy,
  };
}

function loadActiveSources(options = {}) {
  const sourcePath = options.sourcePath || DEFAULT_SOURCE_PATH;
  const log = typeof options.log === 'function' ? options.log : () => {};
  const events = options.events || {};
  const reject = typeof options.reject === 'function' ? options.reject : () => {};
  const sources = loadSources(sourcePath);

  if (!Array.isArray(sources)) {
    throw new Error('Source registry root must be an array.');
  }

  const activeSources = [];

  sources.forEach((source, index) => {
    if (!source || source.active !== true) return;

    const failures = validateSource(source, index).filter((issue) => issue.level === 'FAIL');
    if (failures.length > 0) {
      reject({
        source_id: source && source.source_id ? source.source_id : `sources[${index}]`,
        url: source && source.url ? source.url : '',
        reason: INVALID_SOURCE_REGISTRY,
        details: failures.map((issue) => issue.message),
      });
      log(events.VALIDATION_FAILURE || 'VALIDATION_FAILURE', {
        source_id: source && source.source_id ? source.source_id : null,
        failures: failures.map((issue) => issue.code),
      });
      return;
    }

    activeSources.push(toCrawlerSource(source));
    log(events.SOURCE_LOADED || 'SOURCE_LOADED', { source_id: source.source_id, url: source.url });
  });

  return activeSources;
}

module.exports = {
  DEFAULT_SOURCE_PATH,
  INVALID_SOURCE_REGISTRY,
  loadActiveSources,
};
````

## File: crawler/output/crawl-report.json
````json
{
  "started_at": "2026-06-16T18:24:29.871Z",
  "completed_at": "2026-06-16T18:24:39.475Z",
  "sources_processed": 58,
  "successful_fetches": 0,
  "failed_fetches": 58,
  "staged_records": 0,
  "rejected_records": 58
}
````

## File: crawler/output/fetch-diagnostics.json
````json
{
  "generated_at": "2026-06-16T19:08:57.076Z",
  "input_file": "crawler/output/rejected.json",
  "total_rejected_records": 58,
  "total_fetch_failed_records": 58,
  "categories": {
    "HTTP_403": {
      "count": 58,
      "percentage": 100
    },
    "HTTP_404": {
      "count": 0,
      "percentage": 0
    },
    "HTTP_429": {
      "count": 0,
      "percentage": 0
    },
    "HTTP_5XX": {
      "count": 0,
      "percentage": 0
    },
    "TIMEOUT": {
      "count": 0,
      "percentage": 0
    },
    "DNS_FAILURE": {
      "count": 0,
      "percentage": 0
    },
    "TLS_FAILURE": {
      "count": 0,
      "percentage": 0
    },
    "REDIRECT_FAILURE": {
      "count": 0,
      "percentage": 0
    },
    "UNKNOWN": {
      "count": 0,
      "percentage": 0
    }
  },
  "dominant_failure_category": "HTTP_403"
}
````

## File: crawler/output/rejected.json
````json
[
  {
    "timestamp": "2026-06-16T18:24:30.716Z",
    "source_id": "dpsa",
    "url": "https://www.dpsa.gov.za/newsroom/psvc/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:30.718Z",
    "source_id": "sars",
    "url": "https://www.sars.gov.za/careers/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:30.720Z",
    "source_id": "department-employment-labour",
    "url": "https://www.labour.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:30.722Z",
    "source_id": "national-treasury",
    "url": "https://www.treasury.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:30.723Z",
    "source_id": "department-health",
    "url": "https://www.health.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:31.505Z",
    "source_id": "etdp-seta",
    "url": "https://www.etdpseta.org.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:31.507Z",
    "source_id": "department-basic-education",
    "url": "https://www.education.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:31.509Z",
    "source_id": "bankseta",
    "url": "https://www.bankseta.org.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:31.511Z",
    "source_id": "mict-seta",
    "url": "https://www.mict.org.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:31.512Z",
    "source_id": "services-seta",
    "url": "https://www.serviceseta.org.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:32.301Z",
    "source_id": "ceta",
    "url": "https://www.ceta.org.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:32.304Z",
    "source_id": "teta",
    "url": "https://www.teta.org.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:32.307Z",
    "source_id": "lgseta",
    "url": "https://lgseta.org.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:32.310Z",
    "source_id": "agriseta",
    "url": "https://www.agriseta.co.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:32.313Z",
    "source_id": "merseta",
    "url": "https://www.merseta.org.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:33.095Z",
    "source_id": "hwseta",
    "url": "https://www.hwseta.org.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:33.097Z",
    "source_id": "uj",
    "url": "https://www.uj.ac.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:33.098Z",
    "source_id": "wits",
    "url": "https://www.wits.ac.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:33.100Z",
    "source_id": "unisa",
    "url": "https://www.unisa.ac.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:33.102Z",
    "source_id": "up",
    "url": "https://www.up.ac.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:33.915Z",
    "source_id": "dut",
    "url": "https://www.dut.ac.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:33.918Z",
    "source_id": "cut",
    "url": "https://www.cut.ac.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:33.921Z",
    "source_id": "nmu",
    "url": "https://www.mandela.ac.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:33.924Z",
    "source_id": "ukzn",
    "url": "https://ukzn.ac.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:33.927Z",
    "source_id": "uct",
    "url": "https://www.uct.ac.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:34.716Z",
    "source_id": "tut",
    "url": "https://www.tut.ac.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:34.720Z",
    "source_id": "transnet",
    "url": "https://www.transnet.net/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:34.723Z",
    "source_id": "eskom",
    "url": "https://www.eskom.co.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:34.726Z",
    "source_id": "sanral",
    "url": "https://www.sanral.co.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:34.729Z",
    "source_id": "prasa",
    "url": "https://www.prasa.com/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:35.508Z",
    "source_id": "fnb",
    "url": "https://www.fnb.co.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:35.510Z",
    "source_id": "absa",
    "url": "https://www.absa.africa/absaafrica/careers/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:35.512Z",
    "source_id": "standard-bank",
    "url": "https://www.standardbank.com/sbg/standard-bank-group/careers",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:35.514Z",
    "source_id": "saa",
    "url": "https://www.flysaa.com/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:35.522Z",
    "source_id": "denel",
    "url": "https://www.denel.co.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:36.304Z",
    "source_id": "vodacom",
    "url": "https://www.vodacom.com/careers.php",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:36.306Z",
    "source_id": "mtn",
    "url": "https://group.mtn.com/careers/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:36.308Z",
    "source_id": "nedbank",
    "url": "https://www.nedbank.co.za/content/nedbank/desktop/gt/en/careers.html",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:36.310Z",
    "source_id": "sasol",
    "url": "https://www.sasol.com/careers",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:36.312Z",
    "source_id": "shoprite",
    "url": "https://www.shopriteholdings.co.za/careers.html",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:37.089Z",
    "source_id": "city-of-johannesburg",
    "url": "https://www.joburg.org.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:37.092Z",
    "source_id": "pick-n-pay",
    "url": "https://www.pnp.co.za/careers",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:37.094Z",
    "source_id": "city-of-cape-town",
    "url": "https://www.capetown.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:37.096Z",
    "source_id": "woolworths",
    "url": "https://www.woolworths.co.za/corporate/careers",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:37.098Z",
    "source_id": "ethekwini",
    "url": "https://www.durban.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:37.901Z",
    "source_id": "nelson-mandela-bay",
    "url": "https://www.nelsonmandelabay.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:37.903Z",
    "source_id": "mangaung",
    "url": "https://www.mangaung.co.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:37.905Z",
    "source_id": "tshwane",
    "url": "https://www.tshwane.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:37.906Z",
    "source_id": "ekurhuleni",
    "url": "https://www.ekurhuleni.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:37.908Z",
    "source_id": "buffalo-city",
    "url": "https://www.buffalocity.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:38.686Z",
    "source_id": "stellenbosch-municipality",
    "url": "https://stellenbosch.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:38.689Z",
    "source_id": "msunduzi",
    "url": "https://www.msunduzi.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:38.691Z",
    "source_id": "gauteng-government",
    "url": "https://www.gauteng.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:38.693Z",
    "source_id": "western-cape-government",
    "url": "https://www.westerncape.gov.za/jobs",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:38.694Z",
    "source_id": "kzn-government",
    "url": "https://www.kznonline.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:39.473Z",
    "source_id": "dhet",
    "url": "https://www.dhet.gov.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:39.474Z",
    "source_id": "nsfas",
    "url": "https://www.nsfas.org.za/",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  },
  {
    "timestamp": "2026-06-16T18:24:39.475Z",
    "source_id": "csir",
    "url": "https://www.csir.co.za/careers",
    "reason": "FETCH_FAILED",
    "details": "HTTP 403"
  }
]
````

## File: crawler/output/staging.json
````json
[]
````

## File: crawler/parsers/html-parser.js
````javascript
const cheerio = require('cheerio');
const { normalizeWhitespace } = require('../pipeline/normalize');

function parseHtml(body) {
  const html = Buffer.isBuffer(body) ? body.toString('utf8') : String(body || '');
  const $ = cheerio.load(html);

  $('script, style, noscript, iframe, svg, nav, header, footer, aside').remove();

  const title = normalizeWhitespace($('title').first().text() || $('h1').first().text());
  const links = $('a[href]')
    .map((_, element) => $(element).attr('href'))
    .get()
    .filter(Boolean);
  const headings = $('h1, h2, h3, h4, h5, h6')
    .map((_, element) => normalizeWhitespace($(element).text()))
    .get()
    .filter(Boolean);
  const metadata = {
    title,
    description: normalizeWhitespace($('meta[name="description"]').attr('content') || ''),
    canonical: $('link[rel="canonical"]').attr('href') || '',
  };

  return {
    title,
    text: normalizeWhitespace($('body').text() || $.root().text()),
    links,
    headings,
    metadata,
  };
}

module.exports = {
  parseHtml,
};
````

## File: crawler/parsers/pdf-parser.js
````javascript
const pdfParse = require('pdf-parse');
const { normalizeWhitespace } = require('../pipeline/normalize');

async function parsePdf(body) {
  if (!Buffer.isBuffer(body) || body.length === 0) {
    throw new Error('PDF body is empty.');
  }

  if (body.subarray(0, 5).toString('utf8') !== '%PDF-') {
    throw new Error('PDF body is corrupt or has an invalid signature.');
  }

  try {
    const parsed = await pdfParse(body);
    const text = normalizeWhitespace(parsed.text || '');

    if (!text) {
      throw new Error('PDF did not contain extractable text.');
    }

    return {
      text,
      pages: parsed.numpages || 0,
      info: parsed.info || {},
    };
  } catch (error) {
    const message = /password/i.test(error.message) ? 'PDF is password protected.' : error.message;
    throw new Error(`PDF parsing failed: ${message}`);
  }
}

module.exports = {
  parsePdf,
};
````

## File: crawler/pipeline/normalize.js
````javascript
function normalizeWhitespace(value = '') {
  return String(value)
    .replace(/\r\n/g, '\n')
    .replace(/\r/g, '\n')
    .replace(/\t+/g, ' ')
    .replace(/[ \f\v]+/g, ' ')
    .replace(/ *\n */g, '\n')
    .replace(/\n{3,}/g, '\n\n')
    .trim();
}

function normalizeUrl(value, baseUrl = null) {
  if (!value || typeof value !== 'string') return null;

  try {
    const parsedUrl = baseUrl ? new URL(value, baseUrl) : new URL(value);
    if (!['http:', 'https:'].includes(parsedUrl.protocol)) return null;

    parsedUrl.hash = '';
    parsedUrl.hostname = parsedUrl.hostname.toLowerCase();

    if ((parsedUrl.protocol === 'https:' && parsedUrl.port === '443') || (parsedUrl.protocol === 'http:' && parsedUrl.port === '80')) {
      parsedUrl.port = '';
    }

    return parsedUrl.toString();
  } catch {
    return null;
  }
}

function normalizeLinks(links = [], baseUrl = null) {
  const normalized = links
    .map((link) => normalizeUrl(typeof link === 'string' ? link : link.href, baseUrl))
    .filter(Boolean);

  return [...new Set(normalized)];
}

function normalizeParsedContent(parsed, context = {}) {
  const metadata = {
    ...(parsed.metadata || {}),
    ...(parsed.info ? { info: parsed.info } : {}),
  };

  if (metadata.canonical) {
    metadata.canonical = normalizeUrl(metadata.canonical, context.url);
  }

  if (parsed.title) {
    metadata.title = normalizeWhitespace(parsed.title);
  } else if (metadata.title) {
    metadata.title = normalizeWhitespace(metadata.title);
  }

  if (metadata.description) {
    metadata.description = normalizeWhitespace(metadata.description);
  }

  if (Array.isArray(parsed.links)) {
    metadata.links = normalizeLinks(parsed.links, context.url);
  }

  if (Array.isArray(parsed.headings)) {
    metadata.headings = parsed.headings.map(normalizeWhitespace).filter(Boolean);
  }

  if (Number.isInteger(parsed.pages)) {
    metadata.pages = parsed.pages;
  }

  return {
    normalizedText: normalizeWhitespace(parsed.text || ''),
    metadata,
  };
}

module.exports = {
  normalizeParsedContent,
  normalizeUrl,
  normalizeWhitespace,
};
````

## File: crawler/pipeline/staging.js
````javascript
const fs = require('fs');
const path = require('path');

const OUTPUT_DIR = path.resolve(__dirname, '..', 'output');
const LOG_DIR = path.resolve(__dirname, '..', 'logs');
const STAGING_PATH = path.join(OUTPUT_DIR, 'staging.json');
const REJECTED_PATH = path.join(OUTPUT_DIR, 'rejected.json');
const REPORT_PATH = path.join(OUTPUT_DIR, 'crawl-report.json');
const LOG_PATH = path.join(LOG_DIR, 'crawl.log');

const LOG_EVENTS = Object.freeze({
  START: 'START',
  END: 'END',
  SOURCE_LOADED: 'SOURCE_LOADED',
  FETCH_SUCCESS: 'FETCH_SUCCESS',
  FETCH_FAILURE: 'FETCH_FAILURE',
  UNSUPPORTED_CONTENT: 'UNSUPPORTED_CONTENT',
  PARSE_SUCCESS: 'PARSE_SUCCESS',
  PARSE_FAILURE: 'PARSE_FAILURE',
  VALIDATION_FAILURE: 'VALIDATION_FAILURE',
  STAGED: 'STAGED',
});

function ensureCrawlerStorage() {
  fs.mkdirSync(OUTPUT_DIR, { recursive: true });
  fs.mkdirSync(LOG_DIR, { recursive: true });
}

function writeJson(filePath, payload) {
  ensureCrawlerStorage();
  fs.writeFileSync(filePath, `${JSON.stringify(payload, null, 2)}\n`, 'utf8');
}

function appendLog(event, details = {}) {
  ensureCrawlerStorage();
  const line = JSON.stringify({ timestamp: new Date().toISOString(), event, ...details });
  fs.appendFileSync(LOG_PATH, `${line}\n`, 'utf8');
}

function initializeRunFiles() {
  ensureCrawlerStorage();
  writeJson(STAGING_PATH, []);
  writeJson(REJECTED_PATH, []);
  writeJson(REPORT_PATH, {
    started_at: '',
    completed_at: '',
    sources_processed: 0,
    successful_fetches: 0,
    failed_fetches: 0,
    staged_records: 0,
    rejected_records: 0,
    rejected_by_reason: {},
  });
  fs.writeFileSync(LOG_PATH, '', 'utf8');
}

function stageRecord(records, record) {
  records.push({ ...record, status: 'staged' });
  writeJson(STAGING_PATH, records);
  appendLog(LOG_EVENTS.STAGED, { source_id: record.source_id, url: record.url, crawl_id: record.crawl_id });
}

function rejectRecord(records, rejection) {
  records.push({ timestamp: new Date().toISOString(), ...rejection });
  writeJson(REJECTED_PATH, records);
}

function writeReport(report) {
  writeJson(REPORT_PATH, report);
}

module.exports = {
  LOG_EVENTS,
  LOG_PATH,
  REJECTED_PATH,
  REPORT_PATH,
  STAGING_PATH,
  appendLog,
  initializeRunFiles,
  rejectRecord,
  stageRecord,
  writeReport,
};
````

## File: crawler/pipeline/validate.js
````javascript
function validateNormalizedContent(contentType, normalizedRecord) {
  const reasons = [];
  const title = normalizedRecord.metadata ? normalizedRecord.metadata.title : '';
  const text = normalizedRecord.normalizedText;

  if (contentType === 'html') {
    if (!title || title.trim() === '') reasons.push('HTML_TITLE_MISSING');
    if (!text || text.trim() === '') reasons.push('HTML_TEXT_MISSING');
  } else if (contentType === 'pdf') {
    if (!text || text.trim() === '') reasons.push('PDF_TEXT_MISSING');
  } else {
    reasons.push('UNSUPPORTED_CONTENT_TYPE');
  }

  return {
    valid: reasons.length === 0,
    reasons,
  };
}

module.exports = {
  validateNormalizedContent,
};
````

## File: crawler/reporting/fetch-diagnostics.js
````javascript
const fs = require('fs');
const path = require('path');

const OUTPUT_DIR = path.resolve(__dirname, '..', 'output');
const REJECTED_PATH = path.join(OUTPUT_DIR, 'rejected.json');
const DIAGNOSTICS_PATH = path.join(OUTPUT_DIR, 'fetch-diagnostics.json');

const CATEGORIES = Object.freeze([
  'HTTP_403',
  'HTTP_404',
  'HTTP_429',
  'HTTP_5XX',
  'TIMEOUT',
  'DNS_FAILURE',
  'TLS_FAILURE',
  'REDIRECT_FAILURE',
  'UNKNOWN',
]);

function readJson(filePath) {
  return JSON.parse(fs.readFileSync(filePath, 'utf8'));
}

function classifyFetchFailure(record) {
  const details = String(record.details || '').toLowerCase();

  const statusMatch = details.match(/(?:http|status(?: code)?|response(?: status)?)\D+(\d{3})/i);
  const status = statusMatch ? Number(statusMatch[1]) : null;

  if (status === 403) return 'HTTP_403';
  if (status === 404) return 'HTTP_404';
  if (status === 429) return 'HTTP_429';
  if (status >= 500 && status <= 599) return 'HTTP_5XX';

  if (/\b(etimedout|timeout|timed out|eai_again|socket hang up|aborted)\b/.test(details)) return 'TIMEOUT';
  if (/\b(enotfound|dns|eai_nodata|getaddrinfo|name not resolved)\b/.test(details)) return 'DNS_FAILURE';
  if (/\b(tls|ssl|certificate|cert_|self[- ]signed|unable_to_verify|err_tls|ep roto|wrong version number)\b/.test(details)) return 'TLS_FAILURE';
  if (/\b(redirect|max redirects|too many redirects|redirected|err_fr_too_many_redirects)\b/.test(details)) return 'REDIRECT_FAILURE';

  return 'UNKNOWN';
}

function toPercent(count, total) {
  if (total === 0) return 0;
  return Number(((count / total) * 100).toFixed(2));
}

function buildDiagnostics(rejectedRecords) {
  const fetchFailures = rejectedRecords.filter((record) => record.reason === 'FETCH_FAILED');
  const categories = Object.fromEntries(CATEGORIES.map((category) => [category, { count: 0, percentage: 0 }]));

  for (const record of fetchFailures) {
    categories[classifyFetchFailure(record)].count += 1;
  }

  for (const category of CATEGORIES) {
    categories[category].percentage = toPercent(categories[category].count, fetchFailures.length);
  }

  const dominantFailureCategory = CATEGORIES.reduce((dominant, category) => {
    if (categories[category].count > categories[dominant].count) return category;
    return dominant;
  }, CATEGORIES[0]);

  return {
    generated_at: new Date().toISOString(),
    input_file: path.relative(path.resolve(__dirname, '..', '..'), REJECTED_PATH),
    total_rejected_records: rejectedRecords.length,
    total_fetch_failed_records: fetchFailures.length,
    categories,
    dominant_failure_category: fetchFailures.length > 0 ? dominantFailureCategory : null,
  };
}

function writeDiagnostics(report, outputPath = DIAGNOSTICS_PATH) {
  fs.mkdirSync(path.dirname(outputPath), { recursive: true });
  fs.writeFileSync(outputPath, `${JSON.stringify(report, null, 2)}\n`, 'utf8');
}

function run() {
  const rejectedRecords = readJson(REJECTED_PATH);
  const diagnostics = buildDiagnostics(rejectedRecords);
  writeDiagnostics(diagnostics);
  console.log(`Fetch diagnostics written to ${path.relative(process.cwd(), DIAGNOSTICS_PATH)}`);
  console.log(`Dominant failure category: ${diagnostics.dominant_failure_category}`);
  return diagnostics;
}

if (require.main === module) {
  run();
}

module.exports = {
  CATEGORIES,
  buildDiagnostics,
  classifyFetchFailure,
};
````

## File: crawler/index.js
````javascript
const crypto = require('crypto');
const config = require('./config/crawler-config');
const { loadActiveSources } = require('./core/source-loader');
const { fetchSource } = require('./core/fetcher');
const { routeContent } = require('./core/router');
const { parseHtml } = require('./parsers/html-parser');
const { parsePdf } = require('./parsers/pdf-parser');
const { normalizeParsedContent } = require('./pipeline/normalize');
const { validateNormalizedContent } = require('./pipeline/validate');
const {
  LOG_EVENTS,
  appendLog,
  initializeRunFiles,
  rejectRecord,
  stageRecord,
  writeReport,
} = require('./pipeline/staging');

async function createLimit(concurrency) {
  const { default: pLimit } = await import('p-limit');
  return pLimit(concurrency);
}

function createCrawlId() {
  try {
    const { v4 } = require('uuid');
    return v4();
  } catch {
    return crypto.randomUUID();
  }
}

function reject(rejectedRecords, source, reason, details = null) {
  rejectRecord(rejectedRecords, {
    source_id: source.source_id,
    url: source.url,
    reason,
    ...(details ? { details } : {}),
  });
}

function summarizeRejectionsByReason(rejectedRecords) {
  return rejectedRecords.reduce((summary, record) => {
    const reason = record.reason || 'UNKNOWN';
    summary[reason] = (summary[reason] || 0) + 1;
    return summary;
  }, {});
}

async function parseContent(contentType, body) {
  if (contentType === 'html') return parseHtml(body);
  if (contentType === 'pdf') return parsePdf(body);
  throw new Error(`Unsupported content type: ${contentType}`);
}

async function processSource(source, context) {
  context.report.sources_processed += 1;

  const fetchResult = await fetchSource(source, config, appendLog, LOG_EVENTS);
  if (!fetchResult.ok) {
    context.report.failed_fetches += 1;
    reject(context.rejectedRecords, source, 'FETCH_FAILED', fetchResult.error);
    return;
  }

  context.report.successful_fetches += 1;

  const contentType = routeContent(fetchResult, appendLog, LOG_EVENTS);
  if (contentType === 'unknown') {
    reject(context.rejectedRecords, source, 'UNSUPPORTED_CONTENT_TYPE', fetchResult.contentType || 'No content type detected');
    return;
  }

  let parsed;
  try {
    parsed = await parseContent(contentType, fetchResult.body);
    appendLog(LOG_EVENTS.PARSE_SUCCESS, { source_id: source.source_id, url: fetchResult.url, content_type: contentType });
  } catch (error) {
    appendLog(LOG_EVENTS.PARSE_FAILURE, { source_id: source.source_id, url: fetchResult.url, content_type: contentType, error: error.message });
    reject(context.rejectedRecords, { ...source, url: fetchResult.url }, 'PARSER_ERROR', error.message);
    return;
  }

  const normalized = normalizeParsedContent(parsed, { url: fetchResult.url });
  const validation = validateNormalizedContent(contentType, normalized);
  if (!validation.valid) {
    appendLog(LOG_EVENTS.VALIDATION_FAILURE, { source_id: source.source_id, url: fetchResult.url, reasons: validation.reasons });
    reject(context.rejectedRecords, { ...source, url: fetchResult.url }, 'VALIDATION_FAILED', validation.reasons);
    return;
  }

  const stagedRecord = {
    crawl_id: createCrawlId(),
    source_id: source.source_id,
    source_name: source.name,
    url: fetchResult.url,
    content_type: contentType,
    title: normalized.metadata.title || '',
    text: normalized.normalizedText,
    metadata: normalized.metadata,
    crawled_at: new Date().toISOString(),
  };

  stageRecord(context.stagedRecords, stagedRecord);
}

async function main() {
  initializeRunFiles();

  const report = {
    started_at: new Date().toISOString(),
    completed_at: '',
    sources_processed: 0,
    successful_fetches: 0,
    failed_fetches: 0,
    staged_records: 0,
    rejected_records: 0,
    rejected_by_reason: {},
  };
  const stagedRecords = [];
  const rejectedRecords = [];

  appendLog(LOG_EVENTS.START, { config });

  try {
    const sources = loadActiveSources({
      log: appendLog,
      events: LOG_EVENTS,
      reject: (rejection) => rejectRecord(rejectedRecords, rejection),
    });
    const limit = await createLimit(config.concurrency);

    await Promise.all(
      sources.map((source) => limit(() => processSource(source, { report, stagedRecords, rejectedRecords }))),
    );
  } catch (error) {
    appendLog(LOG_EVENTS.FETCH_FAILURE, { error: error.message, fatal: true });
    process.exitCode = 1;
  } finally {
    report.completed_at = new Date().toISOString();
    report.staged_records = stagedRecords.length;
    report.rejected_records = rejectedRecords.length;
    report.rejected_by_reason = summarizeRejectionsByReason(rejectedRecords);
    writeReport(report);
    appendLog(LOG_EVENTS.END, report);
  }
}

if (require.main === module) {
  main();
}

module.exports = {
  main,
  processSource,
  summarizeRejectionsByReason,
};
````

## File: data/clean/categories.json
````json
[
  {
    "id": "learnership",
    "label": "Learnerships",
    "plural": "learnerships",
    "icon": "🎓",
    "description": "Earn a nationally recognised qualification while earning a monthly stipend. Free to apply.",
    "color": "#166534",
    "bg": "#dcfce7",
    "seo_title": "Learnerships in South Africa 2026 — Find & Apply",
    "seo_desc": "Find verified learnerships across South Africa for 2026. Filter by province, qualification, and closing date. All free to apply. Updated daily."
  },
  {
    "id": "bursary",
    "label": "Bursaries",
    "plural": "bursaries",
    "icon": "📚",
    "description": "Financial funding for studies at a public university or TVET college. Covers tuition and living costs.",
    "color": "#92400e",
    "bg": "#fef3c7",
    "seo_title": "Bursaries South Africa 2026 — Find Funding for Your Studies",
    "seo_desc": "Find government and corporate bursaries for South African students in 2026. NSFAS, merit bursaries, and need-based funding. Updated daily."
  },
  {
    "id": "internship",
    "label": "Internships",
    "plural": "internships",
    "icon": "💼",
    "description": "Structured work experience for graduates and students. Government and private sector programmes.",
    "color": "#1e40af",
    "bg": "#dbeafe",
    "seo_title": "Internships in South Africa 2026 — Graduate Opportunities",
    "seo_desc": "Graduate internships and work experience programmes across South Africa for 2026. Filter by province, sector, and qualification. Verified listings."
  },
  {
    "id": "apprenticeship",
    "label": "Apprenticeships",
    "plural": "apprenticeships",
    "icon": "🔧",
    "description": "Artisan trade training leading to a nationally recognised trade certificate. 2–3 year programmes.",
    "color": "#6b21a8",
    "bg": "#f3e8ff",
    "seo_title": "Apprenticeships in South Africa 2026 — Become a Qualified Artisan",
    "seo_desc": "Find apprenticeship programmes in South Africa for 2026. Electrical, mechanical, boilermaking, plumbing and more. Verified SETA-funded opportunities."
  }
]
````

## File: data/clean/guides.json
````json
[
  {
    "id": "g001",
    "slug": "what-is-a-seta",
    "title": "What Is a SETA in South Africa? Complete Guide 2026",
    "category": "seta-guides",
    "intro": "A SETA — Sector Education and Training Authority — is a government body that funds skills training in South Africa. There are 21 SETAs, each responsible for a different sector of the economy.",
    "body": "<p>SETAs are funded by a 1% payroll levy that every employer with a large enough payroll pays to SARS. This money goes into a Skills Development Fund. SETAs use this fund to pay companies to train unemployed people through learnerships, apprenticeships, and skills programmes.</p><h2>How Do SETAs Help Job Seekers?</h2><p>When a company runs a SETA-funded learnership, they take on unemployed people as learners. You receive on-the-job training AND a nationally recognised qualification — free of charge, with a monthly stipend. You never apply to the SETA directly; you apply to companies running SETA-funded programmes.</p><h2>Which SETA Covers Which Sector?</h2><p>Each SETA covers a specific industry. For IT learnerships, look for MICT SETA programmes. For banking, BANKSETA is relevant. For construction, CETA. For retail, W&RSETA. You'll find the SETA name listed on every opportunity in our directory.</p><h2>Do I Need to Pay Anything?</h2><p><strong>No.</strong> All legitimate SETA-funded learnerships are completely free of charge. If anyone asks you to pay an application fee, registration fee, or any deposit — it is a scam. Report it to the SAPS immediately.</p>",
    "faq": [
      {
        "q": "How many SETAs are there in South Africa?",
        "a": "There are 21 SETAs, each covering a different economic sector."
      },
      {
        "q": "Can I apply directly to a SETA for a learnership?",
        "a": "No. You apply to companies or training providers that run SETA-funded programmes, not to the SETA itself."
      },
      {
        "q": "Are SETA learnerships free?",
        "a": "Yes. All legitimate SETA-funded programmes are completely free. Any programme that charges a fee is a scam."
      }
    ],
    "related_slugs": [
      "what-is-a-learnership",
      "how-to-apply-for-learnerships"
    ],
    "meta_title": "What Is a SETA in South Africa? Complete 2026 Guide",
    "meta_description": "Learn what a SETA is, how SETAs fund learnerships, and how to access SETA opportunities in South Africa. Clear, plain-language guide updated for 2026.",
    "read_time": "4 min",
    "published_date": "2026-04-01",
    "updated_date": "2026-04-20"
  },
  {
    "id": "g002",
    "slug": "what-is-a-learnership",
    "title": "What Is a Learnership in South Africa? 2026 Explained",
    "category": "seta-guides",
    "intro": "A learnership is a structured training programme combining classroom learning with real work experience. You earn a nationally recognised qualification AND a monthly stipend — and it's completely free to apply for.",
    "body": "<p>Learnerships are funded through South Africa's SETA system. When a company runs a learnership, they receive a government grant to cover the cost of training and employing a learner. This is why learnerships exist: government incentivises companies to train people who would otherwise not get work experience.</p><h2>How Long Does a Learnership Last?</h2><p>Most learnerships run for 12 months. Some technical learnerships in engineering, manufacturing, and mining can run 18 to 24 months. At the end, you receive a SAQA-registered NQF qualification.</p><h2>What Is a Learnership Stipend?</h2><p>A stipend is the monthly allowance paid to learners during training. In 2026, most learnerships pay between R3 500 and R8 000 per month, depending on the sector and NQF level. Stipends are not subject to PAYE tax.</p><h2>Who Can Apply?</h2><p>Most learnerships require: South African citizen with a valid ID, Matric certificate, currently unemployed, aged 18–35. Some require specific subjects. Check the eligibility section of each listing carefully.</p>",
    "faq": [
      {
        "q": "How long does a learnership last?",
        "a": "Most learnerships run for 12 months. Technical learnerships may run up to 24 months."
      },
      {
        "q": "Can I do a learnership if I already have a degree?",
        "a": "Yes, but you'll likely qualify for higher-level learnerships or graduate internships instead."
      },
      {
        "q": "Is a learnership the same as an apprenticeship?",
        "a": "No. An apprenticeship leads specifically to a trade certificate (artisan qualification), while a learnership leads to a broader NQF qualification."
      }
    ],
    "related_slugs": [
      "what-is-a-seta",
      "how-to-apply-for-learnerships",
      "nqf-levels-explained"
    ],
    "meta_title": "What Is a Learnership? South Africa Guide 2026",
    "meta_description": "A learnership pays you a monthly stipend while you earn an NQF qualification. Find out how learnerships work, who can apply, and how to get one in South Africa.",
    "read_time": "5 min",
    "published_date": "2026-04-02",
    "updated_date": "2026-04-02"
  },
  {
    "id": "g003",
    "slug": "how-to-apply-for-learnerships",
    "title": "How to Apply for Learnerships: Step-by-Step Guide 2026",
    "category": "seta-guides",
    "intro": "Applying for a learnership doesn't have to be complicated. This step-by-step guide covers everything — from checking your eligibility to submitting your application. Never pay to apply.",
    "body": "<h2>Step 1: Check Your Eligibility</h2><p>Confirm you meet the requirements: South African ID, Matric certificate, unemployed status, and the correct age range. Some learnerships require specific Matric subjects.</p><h2>Step 2: Gather Your Documents</h2><p>You will need: certified copy of your ID (within 3 months), certified Matric certificate, CV of maximum 2 pages, motivational letter of 1 page, and proof of residence not older than 3 months.</p><h2>Step 3: Write a Strong Motivational Letter</h2><p>Most applicants fail at this step by sending a generic letter. Your letter must name the specific company and programme, explain why you want this particular opportunity, and describe what you bring to it. Tailor every letter — generic ones are rejected.</p><h2>Step 4: Submit via the Official Website</h2><p>Apply only through the company's official careers page. Email subject format: <strong>Application: [Learnership Name] — [Your Full Name]</strong>. Send before the closing date — late applications are never considered.</p><h2>Step 5: Follow Up Once</h2><p>One week after applying, send a short follow-up email confirming your application. Once only — multiple follow-ups create a negative impression.</p>",
    "faq": [
      {
        "q": "How many learnerships can I apply for at once?",
        "a": "You can apply for as many as you like, but you can only be enrolled in one at a time."
      },
      {
        "q": "What if I don't have a printer for documents?",
        "a": "Internet cafés and print shops will print and certify your documents for a small fee. The post office offers free certification services."
      }
    ],
    "related_slugs": [
      "what-is-a-learnership",
      "learnership-scams"
    ],
    "meta_title": "How to Apply for a Learnership in South Africa: Step-by-Step 2026",
    "meta_description": "Complete step-by-step guide to applying for learnerships in South Africa. Documents needed, motivational letter tips, and how to find official applications.",
    "read_time": "6 min",
    "published_date": "2026-04-03",
    "updated_date": "2026-04-03"
  },
  {
    "id": "g004",
    "slug": "learnership-scams",
    "title": "How to Avoid Learnership Scams in South Africa (2026)",
    "category": "seta-guides",
    "intro": "Learnership scams are widespread in South Africa. Fake opportunities advertised on social media cost job seekers thousands of rands every year. Here's how to protect yourself.",
    "body": "<p>The most important rule: <strong>legitimate learnerships in South Africa are ALWAYS free to apply for.</strong> Any programme that asks you to pay anything before you start is a scam.</p><h2>7 Warning Signs of a Learnership Scam</h2><p><strong>1. An application fee is requested.</strong> No legitimate SETA-funded learnership ever charges a fee to apply.</p><p><strong>2. Documents requested before an interview.</strong> Real programmes only request documents at the interview stage or after an offer.</p><p><strong>3. The stipend is unusually high.</strong> A learnership advertising R25 000/month is almost certainly fake. Typical stipends are R3 500 to R8 000.</p><p><strong>4. WhatsApp-only contact from an unknown number.</strong> Legitimate employers contact applicants via official company email addresses.</p><p><strong>5. No official company website.</strong> Always verify the company exists at cipc.co.za before engaging.</p><p><strong>6. Urgency pressure.</strong> Apply in the next 24 hours or lose your spot. Legitimate programmes have published closing dates, not artificial urgency.</p><p><strong>7. The email domain is generic.</strong> setalearnerships@gmail.com is not an official company email. Real employers use @companyname.co.za addresses.</p><h2>How to Verify a Legitimate Opportunity</h2><p>Search the company name on Google and go directly to their official website. Look for the opportunity on their official careers page. If it's not there, treat it as suspicious. Verify company registration at cipc.co.za.</p>",
    "faq": [
      {
        "q": "Is it safe to apply via Facebook adverts?",
        "a": "Be very cautious. Always cross-reference the advert with the company's official website before applying or providing any documents."
      },
      {
        "q": "Are all listings on OpportunitiesZA verified?",
        "a": "Yes. Every listing on this site links to the official company or government source. We mark each listing as Verified Source once confirmed."
      }
    ],
    "related_slugs": [
      "what-is-a-learnership",
      "how-to-apply-for-learnerships"
    ],
    "meta_title": "How to Avoid Learnership Scams in South Africa 2026",
    "meta_description": "Learn to identify fake learnerships in South Africa. Know the 7 red flags, how to verify opportunities, and what to do if you've been scammed.",
    "read_time": "5 min",
    "published_date": "2026-04-04",
    "updated_date": "2026-04-04"
  },
  {
    "id": "g005",
    "slug": "nqf-levels-explained",
    "title": "NQF Levels Explained: What They Mean for Your Career (2026)",
    "category": "seta-guides",
    "intro": "The NQF — National Qualifications Framework — classifies all South African qualifications from Level 1 to Level 10. Understanding it helps you know exactly which opportunities you qualify for.",
    "body": "<p>The National Qualifications Framework (NQF) is South Africa's system for classifying all education and training qualifications. Every qualification sits at a specific level. This matters because every opportunity listing states a minimum NQF level or qualification type.</p><h2>The Key Levels</h2><p><strong>NQF Level 4 — Matric (Grade 12 / NSC).</strong> The most common entry level for learnerships. Most SETA learnership programmes lead to an NQF Level 4 qualification.</p><p><strong>NQF Level 5 — Higher Certificate.</strong> A one-year post-Matric qualification. Opens more learnership and some internship doors.</p><p><strong>NQF Level 6 — Diploma / Advanced Certificate.</strong> A three-year TVET or university diploma. Required for many graduate internship programmes.</p><p><strong>NQF Level 7 — Bachelor's Degree.</strong> A three-year university degree. Required for corporate graduate programmes and professional internships.</p><h2>TVET and Learnerships</h2><p>Many SETA learnerships specifically target TVET graduates — particularly N6 graduates who need the 18-month practical component for their National Diploma. A TVET-funded learnership allows you to complete your diploma while being paid a stipend.</p>",
    "faq": [
      {
        "q": "Does a learnership give me an NQF qualification?",
        "a": "Yes. Completing a SETA-registered learnership results in a SAQA-registered qualification at a specific NQF level — most commonly NQF Level 4."
      },
      {
        "q": "Is a TVET diploma equivalent to a university degree?",
        "a": "A 3-year TVET diploma sits at NQF Level 6, while a university degree sits at NQF Level 7. They are different qualifications, but a TVET diploma is widely respected and qualifies you for many graduate opportunities."
      }
    ],
    "related_slugs": [
      "what-is-a-learnership",
      "what-is-a-seta"
    ],
    "meta_title": "NQF Levels Explained — South Africa 2026 Guide",
    "meta_description": "Understand South Africa's NQF qualification levels and what they mean for learnerships, bursaries, and career opportunities. Plain-language guide.",
    "read_time": "4 min",
    "published_date": "2026-04-05",
    "updated_date": "2026-04-05"
  },
  {
    "id": "g006",
    "slug": "nsfas-2026",
    "title": "NSFAS 2026: How to Apply, Who Qualifies & What It Covers",
    "category": "seta-guides",
    "intro": "NSFAS covers tuition, accommodation, and a living allowance for students at public universities and TVET colleges. No repayment required for most students.",
    "body": "<h2>Who Qualifies for NSFAS in 2026?</h2><p>You qualify if: you are a South African citizen, your household income is R350 000 or less per year, and you are registered or planning to register at a public university or TVET college. SASSA grant recipients qualify automatically — no income proof needed.</p><h2>What Does NSFAS Cover?</h2><p>For university students: tuition fees, accommodation (in university residences), meal allowance, transport allowance, and personal care allowance. For TVET college students: tuition and a monthly living allowance.</p><h2>How to Apply</h2><p>Applications open October–November each year at <strong>myNSFAS.org.za</strong> — the only legitimate application portal. Never use agents claiming to apply on your behalf. Any agent charging a fee is running a scam. NSFAS applications are free.</p><h2>NSFAS vs Private Bursary</h2><p>Apply for both — but double funding is not allowed. If you receive a full corporate bursary covering all costs, you cannot also receive NSFAS. Disclose all funding applications honestly on each form.</p>",
    "faq": [
      {
        "q": "What happens if my NSFAS application is rejected?",
        "a": "You can appeal through the myNSFAS portal within the appeal period. Most rejections are due to incomplete applications. Submit all documents correctly the first time."
      },
      {
        "q": "Does NSFAS cover private colleges?",
        "a": "No. NSFAS only covers registered public universities and public TVET colleges."
      }
    ],
    "related_slugs": [
      "what-is-a-learnership",
      "nqf-levels-explained"
    ],
    "meta_title": "NSFAS 2026 Application: Who Qualifies, What It Covers & How to Apply",
    "meta_description": "Everything you need to know about NSFAS 2026 — who qualifies, what is covered, how to apply at myNSFAS, and how it compares to private bursaries.",
    "read_time": "5 min",
    "published_date": "2026-04-06",
    "updated_date": "2026-04-06"
  },
  {
    "id": "g007",
    "slug": "cv-for-learnership",
    "title": "How to Write a CV for a Learnership (With Structure)",
    "category": "career-advice",
    "intro": "Your CV is the first filter between you and a learnership. A clean, 2-page CV with the right information gives you a real advantage over hundreds of applications.",
    "body": "<h2>CV Structure for Learnership Applicants</h2><p><strong>Section 1 — Personal Details:</strong> Full name, ID number, contact number, email address, physical address, province. Do not include a photo unless specifically requested.</p><p><strong>Section 2 — Education:</strong> Institution name, year completed, relevant subjects and symbols. List Matric first. If you have a TVET qualification or any other certificate, list it too.</p><p><strong>Section 3 — Experience:</strong> Any work experience, even informal, part-time, or volunteer. Include company name, your role, dates, and 2–3 bullet points. No experience? Include community work or school projects.</p><p><strong>Section 4 — Skills:</strong> Basic computer literacy, languages spoken, driver's licence if applicable. Keep it brief and honest.</p><p><strong>Section 5 — References:</strong> Two references — a teacher, community leader, or family friend (not a parent). Include name, relationship, and phone number.</p><h2>Non-Negotiable CV Rules</h2><p>Maximum 2 pages. Font: Arial or Calibri, size 11–12. No spelling errors. Save as PDF — not Word (.doc). File name: YourName-CV.pdf. No photo unless requested.</p>",
    "faq": [
      {
        "q": "Should I include a photo on my CV?",
        "a": "Only if the application specifically asks for one. Adding an unrequested photo wastes space."
      },
      {
        "q": "What if I have no work experience at all?",
        "a": "Include volunteer work, helping at a family business, school projects, or community involvement. Something is always better than leaving it blank."
      }
    ],
    "related_slugs": [
      "how-to-apply-for-learnerships",
      "learnership-scams"
    ],
    "meta_title": "How to Write a CV for a Learnership — South Africa 2026",
    "meta_description": "Write a winning CV for learnership applications in South Africa. Structure, rules, and tips — free guide for first-time applicants.",
    "read_time": "5 min",
    "published_date": "2026-04-07",
    "updated_date": "2026-04-07"
  }
]
````

## File: data/clean/opportunities.json
````json
[
  {
    "id": "001",
    "slug": "eskom-engineering-learnership-2026",
    "title": "Eskom Engineering Learnership 2026",
    "category": "learnership",
    "sector": "EWSETA",
    "province": "Nationwide",
    "qualification_level": "Matric",
    "stipend": "R4 500/month",
    "closing_date": "2026-09-30",
    "application_url": "https://www.eskom.co.za/careers",
    "source_name": "Eskom Official Website",
    "source_url": "https://www.eskom.co.za/careers",
    "verified": true,
    "featured": true,
    "expired": false,
    "description": "Eskom's annual learnership programme offers structured workplace learning leading to a nationally recognised NQF qualification. The programme covers electrical, mechanical, and instrumentation disciplines across Eskom's generation, transmission, and distribution divisions.",
    "eligibility": [
      "Grade 12 / Matric certificate",
      "South African citizen with valid ID",
      "Aged 18–35",
      "Currently unemployed",
      "No previous Eskom learnership"
    ],
    "documents_needed": [
      "Certified copy of ID (not older than 3 months)",
      "Certified copy of Matric certificate",
      "Comprehensive CV (max 2 pages)",
      "Proof of residence (not older than 3 months)",
      "Motivational letter (max 1 page)"
    ],
    "tags": [
      "matric",
      "government",
      "nationwide",
      "no-experience"
    ],
    "posted_date": "2026-04-01",
    "updated_date": "2026-04-20"
  },
  {
    "id": "002",
    "slug": "absa-graduate-internship-2026",
    "title": "ABSA Bank Graduate Internship 2026",
    "category": "internship",
    "sector": "BANKSETA",
    "province": "Gauteng",
    "qualification_level": "Degree",
    "stipend": "R8 500/month",
    "closing_date": "2026-07-31",
    "application_url": "https://www.absa.africa/absaafrica/careers",
    "source_name": "ABSA Group Careers",
    "source_url": "https://www.absa.africa/absaafrica/careers",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "12-month structured programme developing young talent in banking and financial services. Interns rotate across Retail, Corporate, and Digital Banking with structured mentorship.",
    "eligibility": [
      "Relevant 3-year degree (Finance, Accounting, IT, Commerce)",
      "South African citizen",
      "Recent graduate — within last 2 years",
      "Aged 18–35"
    ],
    "documents_needed": [
      "Certified ID copy",
      "Certified degree certificate",
      "Academic transcript",
      "CV (max 3 pages)",
      "Cover letter"
    ],
    "tags": [
      "graduate",
      "degree",
      "banking",
      "gauteng"
    ],
    "posted_date": "2026-04-05",
    "updated_date": "2026-04-05"
  },
  {
    "id": "003",
    "slug": "nsfas-bursary-2026",
    "title": "NSFAS Bursary 2026 — University & TVET Funding",
    "category": "bursary",
    "sector": "Government – DHET",
    "province": "Nationwide",
    "qualification_level": "Matric",
    "stipend": "Full bursary – varies by institution",
    "closing_date": "2026-11-30",
    "application_url": "https://www.nsfas.org.za/content/applications.html",
    "source_name": "NSFAS Official Website",
    "source_url": "https://www.nsfas.org.za",
    "verified": true,
    "featured": true,
    "expired": false,
    "description": "NSFAS provides funding to eligible South African students at public universities and TVET colleges. Covers tuition, accommodation, transport, and a living allowance. No repayment required.",
    "eligibility": [
      "South African citizen",
      "Household income below R350 000/year",
      "Registered or applying at a public university or TVET college",
      "SASSA grant recipients qualify automatically"
    ],
    "documents_needed": [
      "South African ID",
      "Proof of income (payslips or affidavit)",
      "Matric results or academic record",
      "Proof of registration at institution"
    ],
    "tags": [
      "government",
      "nationwide",
      "tvet",
      "university",
      "matric"
    ],
    "posted_date": "2026-04-01",
    "updated_date": "2026-04-15"
  },
  {
    "id": "004",
    "slug": "mict-seta-ict-learnership-2026",
    "title": "MICT SETA ICT Learnership 2026",
    "category": "learnership",
    "sector": "MICT SETA",
    "province": "Western Cape",
    "qualification_level": "Matric",
    "stipend": "R3 500/month",
    "closing_date": "2026-08-15",
    "application_url": "https://www.mict.org.za/learnerships",
    "source_name": "MICT SETA Official",
    "source_url": "https://www.mict.org.za",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "NQF Level 4 ICT Learnership providing practical experience in IT support, networking, and software fundamentals over 12 months.",
    "eligibility": [
      "Grade 12 with Mathematics",
      "South African citizen aged 18–35",
      "Basic computer literacy",
      "Currently unemployed"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified Matric certificate",
      "CV",
      "Proof of residence",
      "Motivational letter"
    ],
    "tags": [
      "it",
      "matric",
      "western-cape"
    ],
    "posted_date": "2026-04-08",
    "updated_date": "2026-04-08"
  },
  {
    "id": "005",
    "slug": "transnet-apprenticeship-2026",
    "title": "Transnet Freight Rail Apprenticeship 2026",
    "category": "apprenticeship",
    "sector": "TETA",
    "province": "Gauteng",
    "qualification_level": "Matric",
    "stipend": "R6 000/month",
    "closing_date": "2026-10-31",
    "application_url": "https://www.transnet.net/careers",
    "source_name": "Transnet SOC Ltd",
    "source_url": "https://www.transnet.net/careers",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "3-year apprenticeship developing qualified artisans in diesel mechanical, electrical, and boilermaking trades. Leads to a nationally recognised trade certificate.",
    "eligibility": [
      "Matric with Mathematics and Physical Science (min 50%)",
      "South African citizen aged 18–30",
      "Medically fit (will be tested)"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified Matric with subject results",
      "CV",
      "Medical clearance certificate"
    ],
    "tags": [
      "artisan",
      "maths-science",
      "gauteng",
      "government"
    ],
    "posted_date": "2026-04-10",
    "updated_date": "2026-04-10"
  },
  {
    "id": "006",
    "slug": "shoprite-retail-learnership-2026",
    "title": "Shoprite Group Retail Learnership 2026",
    "category": "learnership",
    "sector": "W&RSETA",
    "province": "Nationwide",
    "qualification_level": "Matric",
    "stipend": "R3 200/month",
    "closing_date": "2026-10-15",
    "application_url": "https://www.shopriteholdings.co.za/careers",
    "source_name": "Shoprite Holdings Careers",
    "source_url": "https://www.shopriteholdings.co.za/careers",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "Retail Learnership across Shoprite, Checkers, and USave providing NQF Level 3 and 4 qualifications in retail operations while working in live store environments.",
    "eligibility": [
      "Grade 12 certificate",
      "South African citizen aged 18–35",
      "Currently unemployed",
      "No previous retail learnership"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified Matric",
      "CV",
      "Motivational letter",
      "Proof of residence"
    ],
    "tags": [
      "retail",
      "nationwide",
      "no-experience",
      "matric"
    ],
    "posted_date": "2026-04-12",
    "updated_date": "2026-04-12"
  },
  {
    "id": "007",
    "slug": "kzn-health-internship-2026",
    "title": "KwaZulu-Natal Department of Health Internship 2026",
    "category": "internship",
    "sector": "PSETA",
    "province": "KwaZulu-Natal",
    "qualification_level": "Degree",
    "stipend": "R7 044/month",
    "closing_date": "2026-09-15",
    "application_url": "https://www.kznhealth.gov.za",
    "source_name": "KZN Department of Health",
    "source_url": "https://www.kznhealth.gov.za",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "12-month internship placements for graduates in health administration, public health, pharmacy assistance, and environmental health at district offices and provincial hospitals.",
    "eligibility": [
      "Relevant health sciences or administration degree",
      "South African citizen aged 18–35",
      "No previous government internship"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified degree certificate",
      "Academic transcript",
      "CV",
      "Z83 form (from dpsa.gov.za)",
      "Cover letter"
    ],
    "tags": [
      "government",
      "health",
      "degree",
      "kwazulu-natal"
    ],
    "posted_date": "2026-04-14",
    "updated_date": "2026-04-14"
  },
  {
    "id": "008",
    "slug": "sasol-bursary-engineering-2026",
    "title": "Sasol Engineering & Science Bursary 2026",
    "category": "bursary",
    "sector": "CHIETA",
    "province": "Nationwide",
    "qualification_level": "Matric",
    "stipend": "Full tuition + R5 000/month",
    "closing_date": "2026-08-31",
    "application_url": "https://www.sasol.com/careers/bursaries",
    "source_name": "Sasol Ltd Careers",
    "source_url": "https://www.sasol.com/careers",
    "verified": true,
    "featured": true,
    "expired": false,
    "description": "Full tuition, accommodation, and monthly living allowance for engineering and science students at any South African public university.",
    "eligibility": [
      "Matric with Maths and Physical Science (min 70%)",
      "South African citizen",
      "Accepted at a South African public university",
      "Studying Chemical, Mechanical, or Electrical Engineering"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified Matric results",
      "University acceptance letter",
      "Motivational letter",
      "CV"
    ],
    "tags": [
      "engineering",
      "science",
      "full-bursary",
      "nationwide"
    ],
    "posted_date": "2026-04-15",
    "updated_date": "2026-04-15"
  },
  {
    "id": "009",
    "slug": "merseta-boilermaker-apprenticeship-2026",
    "title": "MerSETA Boilermaker Apprenticeship 2026",
    "category": "apprenticeship",
    "sector": "MerSETA",
    "province": "Gauteng",
    "qualification_level": "Matric",
    "stipend": "R5 500/month",
    "closing_date": "2026-10-31",
    "application_url": "https://www.merseta.org.za",
    "source_name": "MerSETA Official Website",
    "source_url": "https://www.merseta.org.za",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "3-year boilermaker apprenticeship leading to a nationally recognised trade certificate. Covers metal fabrication, welding, and pressure vessel manufacturing.",
    "eligibility": [
      "Matric with Mathematics and Physical Science (min 50%)",
      "South African citizen aged 18–28",
      "Medically fit"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified Matric with subject results",
      "Medical fitness certificate",
      "CV"
    ],
    "tags": [
      "artisan",
      "boilermaker",
      "maths-science",
      "gauteng"
    ],
    "posted_date": "2026-04-16",
    "updated_date": "2026-04-16"
  },
  {
    "id": "010",
    "slug": "old-mutual-graduate-internship-2026",
    "title": "Old Mutual Graduate Internship 2026",
    "category": "internship",
    "sector": "INSETA",
    "province": "Western Cape",
    "qualification_level": "Degree",
    "stipend": "R9 500/month",
    "closing_date": "2026-07-31",
    "application_url": "https://www.oldmutual.co.za/careers",
    "source_name": "Old Mutual Group Careers",
    "source_url": "https://www.oldmutual.co.za/careers",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "12-month rotational internship across actuarial, finance, IT, and business operations. Structured mentorship with possibility of permanent employment.",
    "eligibility": [
      "BCom, BSc, or relevant 3-year degree",
      "South African citizen aged 18–30",
      "Graduated within last 2 years"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified degree certificate",
      "Academic transcript",
      "CV (max 3 pages)",
      "Cover letter"
    ],
    "tags": [
      "graduate",
      "degree",
      "insurance",
      "western-cape"
    ],
    "posted_date": "2026-04-18",
    "updated_date": "2026-04-18"
  },
  {
    "id": "011",
    "slug": "standard-bank-yes-internship-2026",
    "title": "Standard Bank YES Programme Internship 2026",
    "category": "internship",
    "sector": "BANKSETA",
    "province": "Gauteng",
    "qualification_level": "Degree",
    "stipend": "R6 500/month",
    "closing_date": "2026-09-30",
    "application_url": "https://www.standardbank.co.za/careers",
    "source_name": "Standard Bank Careers Portal",
    "source_url": "https://www.standardbank.co.za/careers",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "12-month YES (Youth Employment Service) placement in banking operations, digital innovation, or financial services. Real work, structured learning, and potential for permanent employment.",
    "eligibility": [
      "Relevant degree (Finance, IT, Commerce, Law)",
      "South African citizen aged 18–35",
      "Unemployed youth"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified degree certificate",
      "CV",
      "Cover letter",
      "Academic transcript"
    ],
    "tags": [
      "graduate",
      "degree",
      "banking",
      "gauteng",
      "yes-programme"
    ],
    "posted_date": "2026-04-20",
    "updated_date": "2026-04-20"
  },
  {
    "id": "012",
    "slug": "capitec-bank-learnership-2026",
    "title": "Capitec Bank Branch Learnership 2026",
    "category": "learnership",
    "sector": "BANKSETA",
    "province": "Western Cape",
    "qualification_level": "Matric",
    "stipend": "R4 000/month",
    "closing_date": "2026-09-30",
    "application_url": "https://www.capitecbank.co.za/careers",
    "source_name": "Capitec Bank Careers",
    "source_url": "https://www.capitecbank.co.za/careers",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "Capitec Bank's branch learnership provides NQF Level 4 retail banking qualification through hands-on customer service training across Capitec branches in the Western Cape.",
    "eligibility": [
      "Matric certificate",
      "South African citizen aged 18–30",
      "Currently unemployed",
      "Passion for customer service"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified Matric",
      "CV",
      "Motivational letter",
      "Proof of residence"
    ],
    "tags": [
      "banking",
      "matric",
      "western-cape",
      "no-experience"
    ],
    "posted_date": "2026-04-22",
    "updated_date": "2026-04-22"
  }
]
````

## File: data/clean/provinces.json
````json
[
  {
    "id": "gauteng",
    "label": "Gauteng",
    "icon": "🏙️",
    "major_cities": [
      "Johannesburg",
      "Pretoria",
      "Ekurhuleni",
      "Soweto"
    ],
    "description": "South Africa's economic heartland. The highest volume of learnerships, internships, and bursaries of any province. Home to the big banks, Eskom, Transnet, and most major corporate headquarters.",
    "top_employers": [
      "Eskom",
      "Transnet",
      "ABSA",
      "Standard Bank",
      "Nedbank",
      "FNB",
      "Old Mutual",
      "Vodacom",
      "MTN",
      "Telkom",
      "Sasol"
    ],
    "seo_title": "Learnerships & Internships in Gauteng 2026 | Johannesburg & Pretoria",
    "seo_desc": "Find verified learnerships, bursaries, and internships in Gauteng for 2026. Johannesburg, Pretoria, and Ekurhuleni. Filter by category and closing date."
  },
  {
    "id": "western-cape",
    "label": "Western Cape",
    "icon": "🌊",
    "major_cities": [
      "Cape Town",
      "Stellenbosch",
      "Paarl",
      "George"
    ],
    "description": "South Africa's second-largest opportunity market. Cape Town hosts major retail headquarters (Shoprite, Pick n Pay, Woolworths), Capitec Bank HQ, and a growing technology sector.",
    "top_employers": [
      "Woolworths",
      "Pick n Pay",
      "Shoprite HQ",
      "Capitec Bank",
      "TFG",
      "Clicks Group",
      "Old Mutual",
      "Sanlam",
      "Remgro"
    ],
    "seo_title": "Learnerships & Internships in Western Cape 2026 | Cape Town",
    "seo_desc": "Find verified learnerships, bursaries, and internships in the Western Cape for 2026. Cape Town and surrounding areas. Verified, free to apply."
  },
  {
    "id": "kwazulu-natal",
    "label": "KwaZulu-Natal",
    "icon": "🌊",
    "major_cities": [
      "Durban",
      "Pietermaritzburg",
      "Richards Bay",
      "Newcastle"
    ],
    "description": "Home to Africa's busiest port, driving consistent Transnet and logistics opportunities. Strong automotive (Toyota SA), sugar, and petrochemicals sectors.",
    "top_employers": [
      "Transnet Port",
      "Toyota SA",
      "Illovo Sugar",
      "Tongaat Hulett",
      "Unilever SA",
      "eThekwini Metro",
      "UKZN"
    ],
    "seo_title": "Learnerships & Internships in KwaZulu-Natal 2026 | Durban",
    "seo_desc": "Find learnerships, bursaries, and internships in KwaZulu-Natal for 2026. Durban, Pietermaritzburg and more. Verified opportunities."
  },
  {
    "id": "eastern-cape",
    "label": "Eastern Cape",
    "icon": "🏔️",
    "major_cities": [
      "Port Elizabeth (Gqeberha)",
      "East London",
      "Mthatha"
    ],
    "description": "South Africa's automotive manufacturing hub. Volkswagen SA (Uitenhage), Isuzu (East London), and Mercedes-Benz SA are major learnership and apprenticeship providers.",
    "top_employers": [
      "Volkswagen SA",
      "Isuzu SA",
      "Mercedes-Benz SA",
      "Nelson Mandela Bay Metro",
      "Buffalo City Metro",
      "Walter Sisulu University"
    ],
    "seo_title": "Learnerships & Internships in Eastern Cape 2026 | Gqeberha & East London",
    "seo_desc": "Find learnerships, bursaries, and internships in the Eastern Cape for 2026. Automotive, government, and manufacturing opportunities. Verified."
  },
  {
    "id": "limpopo",
    "label": "Limpopo",
    "icon": "🌿",
    "major_cities": [
      "Polokwane",
      "Tzaneen",
      "Lephalale",
      "Mokopane"
    ],
    "description": "Mining (platinum, chrome, coal) and agriculture drive the economy. Anglo American Platinum, Implats, and Northam Platinum run consistent learnership and apprenticeship programmes.",
    "top_employers": [
      "Anglo American Platinum",
      "Implats",
      "Northam Platinum",
      "Limpopo Provincial Government",
      "University of Limpopo"
    ],
    "seo_title": "Learnerships & Bursaries in Limpopo 2026 | Polokwane",
    "seo_desc": "Find verified learnerships, bursaries, and internships in Limpopo for 2026. Mining, agriculture, and government opportunities."
  },
  {
    "id": "mpumalanga",
    "label": "Mpumalanga",
    "icon": "⚡",
    "major_cities": [
      "Nelspruit (Mbombela)",
      "Witbank (eMalahleni)",
      "Secunda"
    ],
    "description": "South Africa's energy heartland — home to most of Eskom's power stations and Sasol's Secunda operations. Consistent source of engineering and instrumentation learnership opportunities.",
    "top_employers": [
      "Eskom",
      "Sasol Secunda",
      "Nkangala District Municipality",
      "ACWA Power",
      "Mpumalanga Provincial Government"
    ],
    "seo_title": "Learnerships & Bursaries in Mpumalanga 2026 | Nelspruit & Secunda",
    "seo_desc": "Find verified learnerships and internships in Mpumalanga for 2026. Energy, mining, and government opportunities."
  },
  {
    "id": "north-west",
    "label": "North West",
    "icon": "🌾",
    "major_cities": [
      "Rustenburg",
      "Mahikeng",
      "Klerksdorp"
    ],
    "description": "Centre of South Africa's platinum mining industry. Sibanye-Stillwater, Lonmin, and Anglo American run major learnership and apprenticeship programmes in the Rustenburg area.",
    "top_employers": [
      "Sibanye-Stillwater",
      "Anglo American Platinum",
      "Lonmin",
      "Mahikeng Municipality",
      "North West University"
    ],
    "seo_title": "Learnerships & Bursaries in North West 2026 | Rustenburg",
    "seo_desc": "Find verified learnerships and internships in North West province for 2026. Mining, government, and engineering opportunities."
  },
  {
    "id": "free-state",
    "label": "Free State",
    "icon": "🌻",
    "major_cities": [
      "Bloemfontein",
      "Welkom",
      "Sasolburg"
    ],
    "description": "Agriculture, gold mining near Welkom, and public sector employment in Bloemfontein (judicial capital) drive opportunities. Mangaung Metro is the largest public sector employer.",
    "top_employers": [
      "Mangaung Metro",
      "Central University of Technology",
      "Free State Provincial Government",
      "Sibanye (Welkom)",
      "Free State Department of Health"
    ],
    "seo_title": "Learnerships & Bursaries in Free State 2026 | Bloemfontein",
    "seo_desc": "Find verified learnerships and internships in the Free State for 2026. Government, agriculture, and mining opportunities. Verified listings."
  },
  {
    "id": "northern-cape",
    "label": "Northern Cape",
    "icon": "🏜️",
    "major_cities": [
      "Kimberley",
      "Upington",
      "Springbok"
    ],
    "description": "South Africa's largest but least populated province. Diamond mining (De Beers) and iron ore (Kumba Iron Ore) are primary opportunity sources. Less competition per listing than any other province.",
    "top_employers": [
      "De Beers",
      "Kumba Iron Ore",
      "Assmang Manganese",
      "Sol Plaatje Municipality",
      "Northern Cape Provincial Government"
    ],
    "seo_title": "Learnerships & Bursaries in Northern Cape 2026 | Kimberley",
    "seo_desc": "Find verified learnerships and internships in the Northern Cape for 2026. Mining, government, and artisan opportunities. Less competition."
  }
]
````

## File: data/categories.json
````json
[
  {
    "id": "learnership",
    "label": "Learnerships",
    "plural": "learnerships",
    "icon": "🎓",
    "description": "Earn a nationally recognised qualification while earning a monthly stipend. Free to apply.",
    "color": "#166534",
    "bg": "#dcfce7",
    "seo_title": "Learnerships in South Africa 2026 — Find & Apply",
    "seo_desc": "Find verified learnerships across South Africa for 2026. Filter by province, qualification, and closing date. All free to apply. Updated daily."
  },
  {
    "id": "bursary",
    "label": "Bursaries",
    "plural": "bursaries",
    "icon": "📚",
    "description": "Financial funding for studies at a public university or TVET college. Covers tuition and living costs.",
    "color": "#92400e",
    "bg": "#fef3c7",
    "seo_title": "Bursaries South Africa 2026 — Find Funding for Your Studies",
    "seo_desc": "Find government and corporate bursaries for South African students in 2026. NSFAS, merit bursaries, and need-based funding. Updated daily."
  },
  {
    "id": "internship",
    "label": "Internships",
    "plural": "internships",
    "icon": "💼",
    "description": "Structured work experience for graduates and students. Government and private sector programmes.",
    "color": "#1e40af",
    "bg": "#dbeafe",
    "seo_title": "Internships in South Africa 2026 — Graduate Opportunities",
    "seo_desc": "Graduate internships and work experience programmes across South Africa for 2026. Filter by province, sector, and qualification. Verified listings."
  },
  {
    "id": "apprenticeship",
    "label": "Apprenticeships",
    "plural": "apprenticeships",
    "icon": "🔧",
    "description": "Artisan trade training leading to a nationally recognised trade certificate. 2–3 year programmes.",
    "color": "#6b21a8",
    "bg": "#f3e8ff",
    "seo_title": "Apprenticeships in South Africa 2026 — Become a Qualified Artisan",
    "seo_desc": "Find apprenticeship programmes in South Africa for 2026. Electrical, mechanical, boilermaking, plumbing and more. Verified SETA-funded opportunities."
  }
]
````

## File: data/guides.json
````json
[
  {
    "id": "g001",
    "slug": "what-is-a-seta",
    "title": "What Is a SETA in South Africa? Complete Guide 2026",
    "category": "seta-guides",
    "intro": "A SETA — Sector Education and Training Authority — is a government body that funds skills training in South Africa. There are 21 SETAs, each responsible for a different sector of the economy.",
    "body": "<p>SETAs are funded by a 1% payroll levy that every employer with a large enough payroll pays to SARS. This money goes into a Skills Development Fund. SETAs use this fund to pay companies to train unemployed people through learnerships, apprenticeships, and skills programmes.</p><h2>How Do SETAs Help Job Seekers?</h2><p>When a company runs a SETA-funded learnership, they take on unemployed people as learners. You receive on-the-job training AND a nationally recognised qualification — free of charge, with a monthly stipend. You never apply to the SETA directly; you apply to companies running SETA-funded programmes.</p><h2>Which SETA Covers Which Sector?</h2><p>Each SETA covers a specific industry. For IT learnerships, look for MICT SETA programmes. For banking, BANKSETA is relevant. For construction, CETA. For retail, W&RSETA. You'll find the SETA name listed on every opportunity in our directory.</p><h2>Do I Need to Pay Anything?</h2><p><strong>No.</strong> All legitimate SETA-funded learnerships are completely free of charge. If anyone asks you to pay an application fee, registration fee, or any deposit — it is a scam. Report it to the SAPS immediately.</p>",
    "faq": [
      { "q": "How many SETAs are there in South Africa?", "a": "There are 21 SETAs, each covering a different economic sector." },
      { "q": "Can I apply directly to a SETA for a learnership?", "a": "No. You apply to companies or training providers that run SETA-funded programmes, not to the SETA itself." },
      { "q": "Are SETA learnerships free?", "a": "Yes. All legitimate SETA-funded programmes are completely free. Any programme that charges a fee is a scam." }
    ],
    "related_slugs": ["what-is-a-learnership", "how-to-apply-for-learnerships"],
    "meta_title": "What Is a SETA in South Africa? Complete 2026 Guide",
    "meta_description": "Learn what a SETA is, how SETAs fund learnerships, and how to access SETA opportunities in South Africa. Clear, plain-language guide updated for 2026.",
    "read_time": "4 min",
    "published_date": "2026-04-01",
    "updated_date": "2026-04-20"
  },
  {
    "id": "g002",
    "slug": "what-is-a-learnership",
    "title": "What Is a Learnership in South Africa? 2026 Explained",
    "category": "seta-guides",
    "intro": "A learnership is a structured training programme combining classroom learning with real work experience. You earn a nationally recognised qualification AND a monthly stipend — and it's completely free to apply for.",
    "body": "<p>Learnerships are funded through South Africa's SETA system. When a company runs a learnership, they receive a government grant to cover the cost of training and employing a learner. This is why learnerships exist: government incentivises companies to train people who would otherwise not get work experience.</p><h2>How Long Does a Learnership Last?</h2><p>Most learnerships run for 12 months. Some technical learnerships in engineering, manufacturing, and mining can run 18 to 24 months. At the end, you receive a SAQA-registered NQF qualification.</p><h2>What Is a Learnership Stipend?</h2><p>A stipend is the monthly allowance paid to learners during training. In 2026, most learnerships pay between R3 500 and R8 000 per month, depending on the sector and NQF level. Stipends are not subject to PAYE tax.</p><h2>Who Can Apply?</h2><p>Most learnerships require: South African citizen with a valid ID, Matric certificate, currently unemployed, aged 18–35. Some require specific subjects. Check the eligibility section of each listing carefully.</p>",
    "faq": [
      { "q": "How long does a learnership last?", "a": "Most learnerships run for 12 months. Technical learnerships may run up to 24 months." },
      { "q": "Can I do a learnership if I already have a degree?", "a": "Yes, but you'll likely qualify for higher-level learnerships or graduate internships instead." },
      { "q": "Is a learnership the same as an apprenticeship?", "a": "No. An apprenticeship leads specifically to a trade certificate (artisan qualification), while a learnership leads to a broader NQF qualification." }
    ],
    "related_slugs": ["what-is-a-seta", "how-to-apply-for-learnerships", "nqf-levels-explained"],
    "meta_title": "What Is a Learnership? South Africa Guide 2026",
    "meta_description": "A learnership pays you a monthly stipend while you earn an NQF qualification. Find out how learnerships work, who can apply, and how to get one in South Africa.",
    "read_time": "5 min",
    "published_date": "2026-04-02",
    "updated_date": "2026-04-02"
  },
  {
    "id": "g003",
    "slug": "how-to-apply-for-learnerships",
    "title": "How to Apply for Learnerships: Step-by-Step Guide 2026",
    "category": "seta-guides",
    "intro": "Applying for a learnership doesn't have to be complicated. This step-by-step guide covers everything — from checking your eligibility to submitting your application. Never pay to apply.",
    "body": "<h2>Step 1: Check Your Eligibility</h2><p>Confirm you meet the requirements: South African ID, Matric certificate, unemployed status, and the correct age range. Some learnerships require specific Matric subjects.</p><h2>Step 2: Gather Your Documents</h2><p>You will need: certified copy of your ID (within 3 months), certified Matric certificate, CV of maximum 2 pages, motivational letter of 1 page, and proof of residence not older than 3 months.</p><h2>Step 3: Write a Strong Motivational Letter</h2><p>Most applicants fail at this step by sending a generic letter. Your letter must name the specific company and programme, explain why you want this particular opportunity, and describe what you bring to it. Tailor every letter — generic ones are rejected.</p><h2>Step 4: Submit via the Official Website</h2><p>Apply only through the company's official careers page. Email subject format: <strong>Application: [Learnership Name] — [Your Full Name]</strong>. Send before the closing date — late applications are never considered.</p><h2>Step 5: Follow Up Once</h2><p>One week after applying, send a short follow-up email confirming your application. Once only — multiple follow-ups create a negative impression.</p>",
    "faq": [
      { "q": "How many learnerships can I apply for at once?", "a": "You can apply for as many as you like, but you can only be enrolled in one at a time." },
      { "q": "What if I don't have a printer for documents?", "a": "Internet cafés and print shops will print and certify your documents for a small fee. The post office offers free certification services." }
    ],
    "related_slugs": ["what-is-a-learnership", "learnership-scams"],
    "meta_title": "How to Apply for a Learnership in South Africa: Step-by-Step 2026",
    "meta_description": "Complete step-by-step guide to applying for learnerships in South Africa. Documents needed, motivational letter tips, and how to find official applications.",
    "read_time": "6 min",
    "published_date": "2026-04-03",
    "updated_date": "2026-04-03"
  },
  {
    "id": "g004",
    "slug": "learnership-scams",
    "title": "How to Avoid Learnership Scams in South Africa (2026)",
    "category": "seta-guides",
    "intro": "Learnership scams are widespread in South Africa. Fake opportunities advertised on social media cost job seekers thousands of rands every year. Here's how to protect yourself.",
    "body": "<p>The most important rule: <strong>legitimate learnerships in South Africa are ALWAYS free to apply for.</strong> Any programme that asks you to pay anything before you start is a scam.</p><h2>7 Warning Signs of a Learnership Scam</h2><p><strong>1. An application fee is requested.</strong> No legitimate SETA-funded learnership ever charges a fee to apply.</p><p><strong>2. Documents requested before an interview.</strong> Real programmes only request documents at the interview stage or after an offer.</p><p><strong>3. The stipend is unusually high.</strong> A learnership advertising R25 000/month is almost certainly fake. Typical stipends are R3 500 to R8 000.</p><p><strong>4. WhatsApp-only contact from an unknown number.</strong> Legitimate employers contact applicants via official company email addresses.</p><p><strong>5. No official company website.</strong> Always verify the company exists at cipc.co.za before engaging.</p><p><strong>6. Urgency pressure.</strong> Apply in the next 24 hours or lose your spot. Legitimate programmes have published closing dates, not artificial urgency.</p><p><strong>7. The email domain is generic.</strong> setalearnerships@gmail.com is not an official company email. Real employers use @companyname.co.za addresses.</p><p><strong>8. The application link hides the real employer.</strong> Shortened links, file-sharing forms, or unofficial survey pages are risky unless the same link is published on the employer's official website. Prefer URLs on the employer, SETA, university, or government domain.</p><h2>How to Verify a Legitimate Opportunity</h2><p>Search the company name on Google and go directly to their official website. Look for the opportunity on their official careers page. If it's not there, treat it as suspicious. Verify company registration at cipc.co.za. If a listing is genuine, the application path should be traceable from an official careers, bursary, SETA, university, or government page without paying a fee.</p>",
    "faq": [
      { "q": "Is it safe to apply via Facebook adverts?", "a": "Be very cautious. Always cross-reference the advert with the company's official website before applying or providing any documents." },
      { "q": "Are all listings on OpportunitiesZA verified?", "a": "Yes. Every listing on this site links to the official company or government source. We mark each listing as Verified Source once confirmed." }
    ],
    "related_slugs": ["what-is-a-learnership", "how-to-apply-for-learnerships"],
    "meta_title": "How to Avoid Learnership Scams in South Africa 2026",
    "meta_description": "Learn to identify fake learnerships in South Africa. Know the 7 red flags, how to verify opportunities, and what to do if you've been scammed.",
    "read_time": "5 min",
    "published_date": "2026-04-04",
    "updated_date": "2026-06-03"
  },
  {
    "id": "g005",
    "slug": "nqf-levels-explained",
    "title": "NQF Levels Explained: What They Mean for Your Career (2026)",
    "category": "seta-guides",
    "intro": "The NQF — National Qualifications Framework — classifies all South African qualifications from Level 1 to Level 10. Understanding it helps you know exactly which opportunities you qualify for.",
    "body": "<p>The National Qualifications Framework (NQF) is South Africa's system for classifying all education and training qualifications. Every qualification sits at a specific level. This matters because every opportunity listing states a minimum NQF level or qualification type.</p><h2>The Key Levels</h2><p><strong>NQF Level 4 — Matric (Grade 12 / NSC).</strong> The most common entry level for learnerships. Most SETA learnership programmes lead to an NQF Level 4 qualification.</p><p><strong>NQF Level 5 — Higher Certificate.</strong> A one-year post-Matric qualification. Opens more learnership and some internship doors.</p><p><strong>NQF Level 6 — Diploma / Advanced Certificate.</strong> A three-year TVET or university diploma. Required for many graduate internship programmes.</p><p><strong>NQF Level 7 — Bachelor's Degree.</strong> A three-year university degree. Required for corporate graduate programmes and professional internships.</p><h2>TVET and Learnerships</h2><p>Many SETA learnerships specifically target TVET graduates — particularly N6 graduates who need the 18-month practical component for their National Diploma. A TVET-funded learnership allows you to complete your diploma while being paid a stipend.</p>",
    "faq": [
      { "q": "Does a learnership give me an NQF qualification?", "a": "Yes. Completing a SETA-registered learnership results in a SAQA-registered qualification at a specific NQF level — most commonly NQF Level 4." },
      { "q": "Is a TVET diploma equivalent to a university degree?", "a": "A 3-year TVET diploma sits at NQF Level 6, while a university degree sits at NQF Level 7. They are different qualifications, but a TVET diploma is widely respected and qualifies you for many graduate opportunities." }
    ],
    "related_slugs": ["what-is-a-learnership", "what-is-a-seta"],
    "meta_title": "NQF Levels Explained — South Africa 2026 Guide",
    "meta_description": "Understand South Africa's NQF qualification levels and what they mean for learnerships, bursaries, and career opportunities. Plain-language guide.",
    "read_time": "4 min",
    "published_date": "2026-04-05",
    "updated_date": "2026-04-05"
  },
  {
    "id": "g006",
    "slug": "nsfas-2026",
    "title": "NSFAS 2026: How to Apply, Who Qualifies & What It Covers",
    "category": "seta-guides",
    "intro": "NSFAS covers tuition, accommodation, and a living allowance for students at public universities and TVET colleges. No repayment required for most students.",
    "body": "<h2>Who Qualifies for NSFAS in 2026?</h2><p>You qualify if: you are a South African citizen, your household income is R350 000 or less per year, and you are registered or planning to register at a public university or TVET college. SASSA grant recipients qualify automatically — no income proof needed.</p><h2>What Does NSFAS Cover?</h2><p>For university students: tuition fees, accommodation (in university residences), meal allowance, transport allowance, and personal care allowance. For TVET college students: tuition and a monthly living allowance.</p><h2>How to Apply</h2><p>Applications open October–November each year at <strong>myNSFAS.org.za</strong> — the only legitimate application portal. Never use agents claiming to apply on your behalf. Any agent charging a fee is running a scam. NSFAS applications are free.</p><h2>NSFAS vs Private Bursary</h2><p>Apply for both — but double funding is not allowed. If you receive a full corporate bursary covering all costs, you cannot also receive NSFAS. Disclose all funding applications honestly on each form.</p>",
    "faq": [
      { "q": "What happens if my NSFAS application is rejected?", "a": "You can appeal through the myNSFAS portal within the appeal period. Most rejections are due to incomplete applications. Submit all documents correctly the first time." },
      { "q": "Does NSFAS cover private colleges?", "a": "No. NSFAS only covers registered public universities and public TVET colleges." }
    ],
    "related_slugs": ["what-is-a-learnership", "nqf-levels-explained"],
    "meta_title": "NSFAS 2026 Application: Who Qualifies, What It Covers & How to Apply",
    "meta_description": "Everything you need to know about NSFAS 2026 — who qualifies, what is covered, how to apply at myNSFAS, and how it compares to private bursaries.",
    "read_time": "5 min",
    "published_date": "2026-04-06",
    "updated_date": "2026-04-06"
  },
  {
    "id": "g007",
    "slug": "cv-for-learnership",
    "title": "How to Write a CV for a Learnership (With Structure)",
    "category": "career-advice",
    "intro": "Your CV is the first filter between you and a learnership. A clean, 2-page CV with the right information gives you a real advantage over hundreds of applications.",
    "body": "<h2>CV Structure for Learnership Applicants</h2><p><strong>Section 1 — Personal Details:</strong> Full name, ID number, contact number, email address, physical address, province. Do not include a photo unless specifically requested.</p><p><strong>Section 2 — Education:</strong> Institution name, year completed, relevant subjects and symbols. List Matric first. If you have a TVET qualification or any other certificate, list it too.</p><p><strong>Section 3 — Experience:</strong> Any work experience, even informal, part-time, or volunteer. Include company name, your role, dates, and 2–3 bullet points. No experience? Include community work or school projects.</p><p><strong>Section 4 — Skills:</strong> Basic computer literacy, languages spoken, driver's licence if applicable. Keep it brief and honest.</p><p><strong>Section 5 — References:</strong> Two references — a teacher, community leader, or family friend (not a parent). Include name, relationship, and phone number.</p><h2>Non-Negotiable CV Rules</h2><p>Maximum 2 pages. Font: Arial or Calibri, size 11–12. No spelling errors. Save as PDF — not Word (.doc). File name: YourName-CV.pdf. No photo unless requested.</p>",
    "faq": [
      { "q": "Should I include a photo on my CV?", "a": "Only if the application specifically asks for one. Adding an unrequested photo wastes space." },
      { "q": "What if I have no work experience at all?", "a": "Include volunteer work, helping at a family business, school projects, or community involvement. Something is always better than leaving it blank." }
    ],
    "related_slugs": ["how-to-apply-for-learnerships", "learnership-scams"],
    "meta_title": "How to Write a CV for a Learnership — South Africa 2026",
    "meta_description": "Write a winning CV for learnership applications in South Africa. Structure, rules, and tips — free guide for first-time applicants.",
    "read_time": "5 min",
    "published_date": "2026-04-07",
    "updated_date": "2026-04-07"
  }
]
````

## File: data/opportunities.json
````json
[
  {
    "id": "001",
    "slug": "eskom-engineering-learnership-2026",
    "title": "Eskom Engineering Learnership 2026",
    "category": "learnership",
    "sector": "EWSETA",
    "province": "Nationwide",
    "qualification_level": "Matric",
    "stipend": "R4 500/month",
    "closing_date": "2026-09-30",
    "application_url": "https://www.eskom.co.za/careers",
    "source_name": "Eskom Official Website",
    "source_url": "https://www.eskom.co.za/careers",
    "verified": true,
    "featured": true,
    "expired": false,
    "description": "Refreshed official-source listing for Eskom engineering learnership pathways. Applicants should monitor Eskom's official careers portal for advertised learner roles in electrical, mechanical, and instrumentation disciplines across generation, transmission, and distribution teams.",
    "eligibility": [
      "Grade 12 / Matric certificate",
      "South African citizen with valid ID",
      "Aged 18–35",
      "Currently unemployed",
      "No previous Eskom learnership"
    ],
    "documents_needed": [
      "Certified copy of ID (not older than 3 months)",
      "Certified copy of Matric certificate",
      "Comprehensive CV (max 2 pages)",
      "Proof of residence (not older than 3 months)",
      "Motivational letter (max 1 page)"
    ],
    "tags": ["matric", "government", "nationwide", "no-experience", "official-source-refreshed"],
    "posted_date": "2026-04-01",
    "updated_date": "2026-06-03"
  },
  {
    "id": "002",
    "slug": "absa-graduate-internship-2026",
    "title": "ABSA Bank Graduate Internship 2026",
    "category": "internship",
    "sector": "BANKSETA",
    "province": "Gauteng",
    "qualification_level": "Degree",
    "stipend": "R8 500/month",
    "closing_date": "2026-07-31",
    "application_url": "https://www.absa.africa/absaafrica/careers",
    "source_name": "ABSA Group Careers",
    "source_url": "https://www.absa.africa/absaafrica/careers",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "12-month structured programme developing young talent in banking and financial services. Interns rotate across Retail, Corporate, and Digital Banking with structured mentorship.",
    "eligibility": [
      "Relevant 3-year degree (Finance, Accounting, IT, Commerce)",
      "South African citizen",
      "Recent graduate — within last 2 years",
      "Aged 18–35"
    ],
    "documents_needed": [
      "Certified ID copy",
      "Certified degree certificate",
      "Academic transcript",
      "CV (max 3 pages)",
      "Cover letter"
    ],
    "tags": ["graduate", "degree", "banking", "gauteng"],
    "posted_date": "2026-04-05",
    "updated_date": "2026-04-05"
  },
  {
    "id": "003",
    "slug": "nsfas-bursary-2026",
    "title": "NSFAS Bursary 2026 — University & TVET Funding",
    "category": "bursary",
    "sector": "Government – DHET",
    "province": "Nationwide",
    "qualification_level": "Matric",
    "stipend": "Full bursary – varies by institution",
    "closing_date": "2026-11-30",
    "application_url": "https://my.nsfas.org.za",
    "source_name": "NSFAS Official Website",
    "source_url": "https://www.nsfas.org.za",
    "verified": true,
    "featured": true,
    "expired": false,
    "description": "Refreshed official-source NSFAS funding listing for South African students planning to study at public universities or TVET colleges. Funding support can include tuition, accommodation, transport, meals, and living allowances according to NSFAS rules.",
    "eligibility": [
      "South African citizen",
      "Household income below R350 000/year",
      "Registered or applying at a public university or TVET college",
      "SASSA grant recipients qualify automatically"
    ],
    "documents_needed": [
      "South African ID",
      "Proof of income (payslips or affidavit)",
      "Matric results or academic record",
      "Proof of registration at institution"
    ],
    "tags": ["government", "nationwide", "tvet", "university", "matric", "official-source-refreshed"],
    "posted_date": "2026-04-01",
    "updated_date": "2026-06-03"
  },
  {
    "id": "004",
    "slug": "mict-seta-ict-learnership-2026",
    "title": "MICT SETA ICT Learnership 2026",
    "category": "learnership",
    "sector": "MICT SETA",
    "province": "Western Cape",
    "qualification_level": "Matric",
    "stipend": "R3 500/month",
    "closing_date": "2026-08-15",
    "application_url": "https://www.mict.org.za/learnerships",
    "source_name": "MICT SETA Official",
    "source_url": "https://www.mict.org.za",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "NQF Level 4 ICT Learnership providing practical experience in IT support, networking, and software fundamentals over 12 months.",
    "eligibility": [
      "Grade 12 with Mathematics",
      "South African citizen aged 18–35",
      "Basic computer literacy",
      "Currently unemployed"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified Matric certificate",
      "CV",
      "Proof of residence",
      "Motivational letter"
    ],
    "tags": ["it", "matric", "western-cape"],
    "posted_date": "2026-04-08",
    "updated_date": "2026-04-08"
  },
  {
    "id": "005",
    "slug": "transnet-apprenticeship-2026",
    "title": "Transnet Freight Rail Apprenticeship 2026",
    "category": "apprenticeship",
    "sector": "TETA",
    "province": "Gauteng",
    "qualification_level": "Matric",
    "stipend": "R6 000/month",
    "closing_date": "2026-10-31",
    "application_url": "https://www.transnet.net/careers",
    "source_name": "Transnet SOC Ltd",
    "source_url": "https://www.transnet.net/careers",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "3-year apprenticeship developing qualified artisans in diesel mechanical, electrical, and boilermaking trades. Leads to a nationally recognised trade certificate.",
    "eligibility": [
      "Matric with Mathematics and Physical Science (min 50%)",
      "South African citizen aged 18–30",
      "Medically fit (will be tested)"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified Matric with subject results",
      "CV",
      "Medical clearance certificate"
    ],
    "tags": ["artisan", "maths-science", "gauteng", "government"],
    "posted_date": "2026-04-10",
    "updated_date": "2026-04-10"
  },
  {
    "id": "006",
    "slug": "shoprite-retail-learnership-2026",
    "title": "Shoprite Group Retail Learnership 2026",
    "category": "learnership",
    "sector": "W&RSETA",
    "province": "Nationwide",
    "qualification_level": "Matric",
    "stipend": "R3 200/month",
    "closing_date": "2026-10-15",
    "application_url": "https://www.shopriteholdings.co.za/careers",
    "source_name": "Shoprite Holdings Careers",
    "source_url": "https://www.shopriteholdings.co.za/careers",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "Retail Learnership across Shoprite, Checkers, and USave providing NQF Level 3 and 4 qualifications in retail operations while working in live store environments.",
    "eligibility": [
      "Grade 12 certificate",
      "South African citizen aged 18–35",
      "Currently unemployed",
      "No previous retail learnership"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified Matric",
      "CV",
      "Motivational letter",
      "Proof of residence"
    ],
    "tags": ["retail", "nationwide", "no-experience", "matric"],
    "posted_date": "2026-04-12",
    "updated_date": "2026-04-12"
  },
  {
    "id": "007",
    "slug": "kzn-health-internship-2026",
    "title": "KwaZulu-Natal Department of Health Internship 2026",
    "category": "internship",
    "sector": "PSETA",
    "province": "KwaZulu-Natal",
    "qualification_level": "Degree",
    "stipend": "R7 044/month",
    "closing_date": "2026-09-15",
    "application_url": "https://www.kznhealth.gov.za",
    "source_name": "KZN Department of Health",
    "source_url": "https://www.kznhealth.gov.za",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "12-month internship placements for graduates in health administration, public health, pharmacy assistance, and environmental health at district offices and provincial hospitals.",
    "eligibility": [
      "Relevant health sciences or administration degree",
      "South African citizen aged 18–35",
      "No previous government internship"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified degree certificate",
      "Academic transcript",
      "CV",
      "Z83 form (from dpsa.gov.za)",
      "Cover letter"
    ],
    "tags": ["government", "health", "degree", "kwazulu-natal"],
    "posted_date": "2026-04-14",
    "updated_date": "2026-04-14"
  },
  {
    "id": "008",
    "slug": "sasol-bursary-engineering-2026",
    "title": "Sasol Engineering & Science Bursary 2026",
    "category": "bursary",
    "sector": "CHIETA",
    "province": "Nationwide",
    "qualification_level": "Matric",
    "stipend": "Full tuition + R5 000/month",
    "closing_date": "2026-08-31",
    "application_url": "https://www.sasolbursaries.com/welcome/how-to-apply/",
    "source_name": "Sasol Bursaries",
    "source_url": "https://www.sasolbursaries.com/welcome/",
    "verified": true,
    "featured": true,
    "expired": false,
    "description": "Refreshed official-source Sasol bursary listing for South African students pursuing engineering, science, and technology-related studies. Applicants should use the Sasol Bursaries portal for current eligibility rules, application steps, and programme updates.",
    "eligibility": [
      "Matric with Maths and Physical Science (min 70%)",
      "South African citizen",
      "Accepted at a South African public university",
      "Studying Chemical, Mechanical, or Electrical Engineering"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified Matric results",
      "University acceptance letter",
      "Motivational letter",
      "CV"
    ],
    "tags": ["engineering", "science", "full-bursary", "stem", "nationwide", "official-source-refreshed"],
    "posted_date": "2026-04-15",
    "updated_date": "2026-06-03"
  },
  {
    "id": "009",
    "slug": "merseta-boilermaker-apprenticeship-2026",
    "title": "MerSETA Boilermaker Apprenticeship 2026",
    "category": "apprenticeship",
    "sector": "MerSETA",
    "province": "Gauteng",
    "qualification_level": "Matric",
    "stipend": "R5 500/month",
    "closing_date": "2026-10-31",
    "application_url": "https://www.merseta.org.za",
    "source_name": "MerSETA Official Website",
    "source_url": "https://www.merseta.org.za",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "3-year boilermaker apprenticeship leading to a nationally recognised trade certificate. Covers metal fabrication, welding, and pressure vessel manufacturing.",
    "eligibility": [
      "Matric with Mathematics and Physical Science (min 50%)",
      "South African citizen aged 18–28",
      "Medically fit"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified Matric with subject results",
      "Medical fitness certificate",
      "CV"
    ],
    "tags": ["artisan", "boilermaker", "maths-science", "gauteng"],
    "posted_date": "2026-04-16",
    "updated_date": "2026-04-16"
  },
  {
    "id": "010",
    "slug": "old-mutual-graduate-internship-2026",
    "title": "Old Mutual Graduate Internship 2026",
    "category": "internship",
    "sector": "INSETA",
    "province": "Western Cape",
    "qualification_level": "Degree",
    "stipend": "R9 500/month",
    "closing_date": "2026-07-31",
    "application_url": "https://www.oldmutual.co.za/careers",
    "source_name": "Old Mutual Group Careers",
    "source_url": "https://www.oldmutual.co.za/careers",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "12-month rotational internship across actuarial, finance, IT, and business operations. Structured mentorship with possibility of permanent employment.",
    "eligibility": [
      "BCom, BSc, or relevant 3-year degree",
      "South African citizen aged 18–30",
      "Graduated within last 2 years"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified degree certificate",
      "Academic transcript",
      "CV (max 3 pages)",
      "Cover letter"
    ],
    "tags": ["graduate", "degree", "insurance", "western-cape"],
    "posted_date": "2026-04-18",
    "updated_date": "2026-04-18"
  },
  {
    "id": "011",
    "slug": "standard-bank-yes-internship-2026",
    "title": "Standard Bank YES Programme Internship 2026",
    "category": "internship",
    "sector": "BANKSETA",
    "province": "Gauteng",
    "qualification_level": "Degree",
    "stipend": "R6 500/month",
    "closing_date": "2026-09-30",
    "application_url": "https://www.standardbank.co.za/careers",
    "source_name": "Standard Bank Careers Portal",
    "source_url": "https://www.standardbank.co.za/careers",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "12-month YES (Youth Employment Service) placement in banking operations, digital innovation, or financial services. Real work, structured learning, and potential for permanent employment.",
    "eligibility": [
      "Relevant degree (Finance, IT, Commerce, Law)",
      "South African citizen aged 18–35",
      "Unemployed youth"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified degree certificate",
      "CV",
      "Cover letter",
      "Academic transcript"
    ],
    "tags": ["graduate", "degree", "banking", "gauteng", "yes-programme"],
    "posted_date": "2026-04-20",
    "updated_date": "2026-04-20"
  },
  {
    "id": "012",
    "slug": "capitec-bank-learnership-2026",
    "title": "Capitec Bank Branch Learnership 2026",
    "category": "learnership",
    "sector": "BANKSETA",
    "province": "Western Cape",
    "qualification_level": "Matric",
    "stipend": "R4 000/month",
    "closing_date": "2026-09-30",
    "application_url": "https://www.capitecbank.co.za/careers",
    "source_name": "Capitec Bank Careers",
    "source_url": "https://www.capitecbank.co.za/careers",
    "verified": true,
    "featured": false,
    "expired": false,
    "description": "Refreshed official-source listing for Capitec entry-level branch and banking learning pathways. Applicants should monitor Capitec's official careers portal for branch consultant, service, and learnership-style opportunities that build retail banking and customer-service skills.",
    "eligibility": [
      "Matric certificate",
      "South African citizen aged 18–30",
      "Currently unemployed",
      "Passion for customer service"
    ],
    "documents_needed": [
      "Certified ID",
      "Certified Matric",
      "CV",
      "Motivational letter",
      "Proof of residence"
    ],
    "tags": ["banking", "matric", "western-cape", "no-experience", "official-source-refreshed"],
    "posted_date": "2026-04-22",
    "updated_date": "2026-06-03"
  }
]
````

## File: data/provinces.json
````json
[
  {
    "id": "gauteng",
    "label": "Gauteng",
    "icon": "🏙️",
    "major_cities": ["Johannesburg", "Pretoria", "Ekurhuleni", "Soweto"],
    "description": "South Africa's economic heartland. The highest volume of learnerships, internships, and bursaries of any province. Home to the big banks, Eskom, Transnet, and most major corporate headquarters.",
    "top_employers": ["Eskom", "Transnet", "ABSA", "Standard Bank", "Nedbank", "FNB", "Old Mutual", "Vodacom", "MTN", "Telkom", "Sasol"],
    "seo_title": "Learnerships & Internships in Gauteng 2026 | Johannesburg & Pretoria",
    "seo_desc": "Find verified learnerships, bursaries, and internships in Gauteng for 2026. Johannesburg, Pretoria, and Ekurhuleni. Filter by category and closing date."
  },
  {
    "id": "western-cape",
    "label": "Western Cape",
    "icon": "🌊",
    "major_cities": ["Cape Town", "Stellenbosch", "Paarl", "George"],
    "description": "South Africa's second-largest opportunity market. Cape Town hosts major retail headquarters (Shoprite, Pick n Pay, Woolworths), Capitec Bank HQ, and a growing technology sector.",
    "top_employers": ["Woolworths", "Pick n Pay", "Shoprite HQ", "Capitec Bank", "TFG", "Clicks Group", "Old Mutual", "Sanlam", "Remgro"],
    "seo_title": "Learnerships & Internships in Western Cape 2026 | Cape Town",
    "seo_desc": "Find verified learnerships, bursaries, and internships in the Western Cape for 2026. Cape Town and surrounding areas. Verified, free to apply."
  },
  {
    "id": "kwazulu-natal",
    "label": "KwaZulu-Natal",
    "icon": "🌊",
    "major_cities": ["Durban", "Pietermaritzburg", "Richards Bay", "Newcastle"],
    "description": "Home to Africa's busiest port, driving consistent Transnet and logistics opportunities. Strong automotive (Toyota SA), sugar, and petrochemicals sectors.",
    "top_employers": ["Transnet Port", "Toyota SA", "Illovo Sugar", "Tongaat Hulett", "Unilever SA", "eThekwini Metro", "UKZN"],
    "seo_title": "Learnerships & Internships in KwaZulu-Natal 2026 | Durban",
    "seo_desc": "Find learnerships, bursaries, and internships in KwaZulu-Natal for 2026. Durban, Pietermaritzburg and more. Verified opportunities."
  },
  {
    "id": "eastern-cape",
    "label": "Eastern Cape",
    "icon": "🏔️",
    "major_cities": ["Port Elizabeth (Gqeberha)", "East London", "Mthatha"],
    "description": "South Africa's automotive manufacturing hub. Volkswagen SA (Uitenhage), Isuzu (East London), and Mercedes-Benz SA are major learnership and apprenticeship providers.",
    "top_employers": ["Volkswagen SA", "Isuzu SA", "Mercedes-Benz SA", "Nelson Mandela Bay Metro", "Buffalo City Metro", "Walter Sisulu University"],
    "seo_title": "Learnerships & Internships in Eastern Cape 2026 | Gqeberha & East London",
    "seo_desc": "Find learnerships, bursaries, and internships in the Eastern Cape for 2026. Automotive, government, and manufacturing opportunities. Verified."
  },
  {
    "id": "limpopo",
    "label": "Limpopo",
    "icon": "🌿",
    "major_cities": ["Polokwane", "Tzaneen", "Lephalale", "Mokopane"],
    "description": "Mining (platinum, chrome, coal) and agriculture drive the economy. Anglo American Platinum, Implats, and Northam Platinum run consistent learnership and apprenticeship programmes.",
    "top_employers": ["Anglo American Platinum", "Implats", "Northam Platinum", "Limpopo Provincial Government", "University of Limpopo"],
    "seo_title": "Learnerships & Bursaries in Limpopo 2026 | Polokwane",
    "seo_desc": "Find verified learnerships, bursaries, and internships in Limpopo for 2026. Mining, agriculture, and government opportunities."
  },
  {
    "id": "mpumalanga",
    "label": "Mpumalanga",
    "icon": "⚡",
    "major_cities": ["Nelspruit (Mbombela)", "Witbank (eMalahleni)", "Secunda"],
    "description": "South Africa's energy heartland — home to most of Eskom's power stations and Sasol's Secunda operations. Consistent source of engineering and instrumentation learnership opportunities.",
    "top_employers": ["Eskom", "Sasol Secunda", "Nkangala District Municipality", "ACWA Power", "Mpumalanga Provincial Government"],
    "seo_title": "Learnerships & Bursaries in Mpumalanga 2026 | Nelspruit & Secunda",
    "seo_desc": "Find verified learnerships and internships in Mpumalanga for 2026. Energy, mining, and government opportunities."
  },
  {
    "id": "north-west",
    "label": "North West",
    "icon": "🌾",
    "major_cities": ["Rustenburg", "Mahikeng", "Klerksdorp"],
    "description": "Centre of South Africa's platinum mining industry. Sibanye-Stillwater, Lonmin, and Anglo American run major learnership and apprenticeship programmes in the Rustenburg area.",
    "top_employers": ["Sibanye-Stillwater", "Anglo American Platinum", "Lonmin", "Mahikeng Municipality", "North West University"],
    "seo_title": "Learnerships & Bursaries in North West 2026 | Rustenburg",
    "seo_desc": "Find verified learnerships and internships in North West province for 2026. Mining, government, and engineering opportunities."
  },
  {
    "id": "free-state",
    "label": "Free State",
    "icon": "🌻",
    "major_cities": ["Bloemfontein", "Welkom", "Sasolburg"],
    "description": "Agriculture, gold mining near Welkom, and public sector employment in Bloemfontein (judicial capital) drive opportunities. Mangaung Metro is the largest public sector employer.",
    "top_employers": ["Mangaung Metro", "Central University of Technology", "Free State Provincial Government", "Sibanye (Welkom)", "Free State Department of Health"],
    "seo_title": "Learnerships & Bursaries in Free State 2026 | Bloemfontein",
    "seo_desc": "Find verified learnerships and internships in the Free State for 2026. Government, agriculture, and mining opportunities. Verified listings."
  },
  {
    "id": "northern-cape",
    "label": "Northern Cape",
    "icon": "🏜️",
    "major_cities": ["Kimberley", "Upington", "Springbok"],
    "description": "South Africa's largest but least populated province. Diamond mining (De Beers) and iron ore (Kumba Iron Ore) are primary opportunity sources. Less competition per listing than any other province.",
    "top_employers": ["De Beers", "Kumba Iron Ore", "Assmang Manganese", "Sol Plaatje Municipality", "Northern Cape Provincial Government"],
    "seo_title": "Learnerships & Bursaries in Northern Cape 2026 | Kimberley",
    "seo_desc": "Find verified learnerships and internships in the Northern Cape for 2026. Mining, government, and artisan opportunities. Less competition."
  }
]
````

## File: data/sources.json
````json
[
  {"source_id":"dpsa","name":"Department of Public Service and Administration (DPSA)","type":"government","url":"https://www.dpsa.gov.za/newsroom/psvc/","active":true,"crawl_strategy":"pdf","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Public Service Vacancy Circulars."},
  {"source_id":"national-treasury","name":"National Treasury","type":"government","url":"https://www.treasury.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Government finance jobs, internships, bursaries and tender-related notices."},
  {"source_id":"sars","name":"South African Revenue Service (SARS)","type":"government","url":"https://www.sars.gov.za/careers/","active":true,"crawl_strategy":"html","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Tax administration careers and graduate opportunities."},
  {"source_id":"department-employment-labour","name":"Department of Employment and Labour","type":"government","url":"https://www.labour.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Public employment services, departmental vacancies and labour market programmes."},
  {"source_id":"department-health","name":"National Department of Health","type":"government","url":"https://www.health.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Health-sector vacancies, internships and programme notices."},
  {"source_id":"department-basic-education","name":"Department of Basic Education","type":"government","url":"https://www.education.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Education department vacancies, bursaries and programme announcements."},
  {"source_id":"mict-seta","name":"MICT SETA","type":"seta","url":"https://www.mict.org.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"ICT, media and telecommunications skills opportunities."},
  {"source_id":"services-seta","name":"Services SETA","type":"seta","url":"https://www.serviceseta.org.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Services-sector learnerships, discretionary grants and skills programmes."},
  {"source_id":"etdp-seta","name":"ETDP SETA","type":"seta","url":"https://www.etdpseta.org.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Education, training and development opportunities."},
  {"source_id":"bankseta","name":"BANKSETA","type":"seta","url":"https://www.bankseta.org.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Banking-sector skills programmes, learnerships and bursaries."},
  {"source_id":"agriseta","name":"AGRISETA","type":"seta","url":"https://www.agriseta.co.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Agriculture-sector skills and grant opportunities."},
  {"source_id":"ceta","name":"Construction Education and Training Authority (CETA)","type":"seta","url":"https://www.ceta.org.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Construction-sector learnerships, bursaries and tenders."},
  {"source_id":"merseta","name":"merSETA","type":"seta","url":"https://www.merseta.org.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Manufacturing, engineering and related services skills opportunities."},
  {"source_id":"lgseta","name":"LGSETA","type":"seta","url":"https://lgseta.org.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Local government skills programmes and discretionary grants."},
  {"source_id":"teta","name":"Transport Education Training Authority (TETA)","type":"seta","url":"https://www.teta.org.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Transport-sector bursaries, learnerships and skills programmes."},
  {"source_id":"hwseta","name":"HWSETA","type":"seta","url":"https://www.hwseta.org.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Health and welfare-sector skills development opportunities."},
  {"source_id":"unisa","name":"University of South Africa (UNISA)","type":"university","url":"https://www.unisa.ac.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"University vacancies, student funding and graduate opportunities."},
  {"source_id":"uj","name":"University of Johannesburg (UJ)","type":"university","url":"https://www.uj.ac.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"University vacancies, internships and bursary notices."},
  {"source_id":"wits","name":"University of the Witwatersrand (Wits)","type":"university","url":"https://www.wits.ac.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Academic, professional and student opportunity notices."},
  {"source_id":"up","name":"University of Pretoria (UP)","type":"university","url":"https://www.up.ac.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"University careers, bursaries and graduate programmes."},
  {"source_id":"uct","name":"University of Cape Town (UCT)","type":"university","url":"https://www.uct.ac.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"University vacancies and student funding opportunities."},
  {"source_id":"ukzn","name":"University of KwaZulu-Natal (UKZN)","type":"university","url":"https://ukzn.ac.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"University careers and student opportunity notices."},
  {"source_id":"nmu","name":"Nelson Mandela University (NMU)","type":"university","url":"https://www.mandela.ac.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":3,"last_checked":null,"last_success":null,"notes":"University vacancies, internships and bursaries."},
  {"source_id":"dut","name":"Durban University of Technology (DUT)","type":"university","url":"https://www.dut.ac.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":3,"last_checked":null,"last_success":null,"notes":"University careers and student opportunities."},
  {"source_id":"cut","name":"Central University of Technology (CUT)","type":"university","url":"https://www.cut.ac.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":3,"last_checked":null,"last_success":null,"notes":"University vacancies and student support opportunities."},
  {"source_id":"tut","name":"Tshwane University of Technology (TUT)","type":"university","url":"https://www.tut.ac.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"University vacancies, bursaries and internships."},
  {"source_id":"eskom","name":"Eskom","type":"soe","url":"https://www.eskom.co.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Energy-sector careers, apprenticeships and graduate programmes."},
  {"source_id":"transnet","name":"Transnet","type":"soe","url":"https://www.transnet.net/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Transport and logistics careers, learnerships and bursaries."},
  {"source_id":"prasa","name":"Passenger Rail Agency of South Africa (PRASA)","type":"soe","url":"https://www.prasa.com/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Rail-sector vacancies and internship opportunities."},
  {"source_id":"sanral","name":"South African National Roads Agency (SANRAL)","type":"soe","url":"https://www.sanral.co.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Roads infrastructure careers, bursaries and scholarships."},
  {"source_id":"saa","name":"South African Airways (SAA)","type":"soe","url":"https://www.flysaa.com/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":3,"last_checked":null,"last_success":null,"notes":"Aviation careers and trainee opportunities."},
  {"source_id":"denel","name":"Denel","type":"soe","url":"https://www.denel.co.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":3,"last_checked":null,"last_success":null,"notes":"Defence manufacturing careers and technical opportunities."},
  {"source_id":"standard-bank","name":"Standard Bank","type":"private","url":"https://www.standardbank.com/sbg/standard-bank-group/careers","active":true,"crawl_strategy":"html","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Banking careers, internships and graduate programmes."},
  {"source_id":"fnb","name":"First National Bank (FNB)","type":"private","url":"https://www.fnb.co.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Banking and fintech career opportunities."},
  {"source_id":"absa","name":"Absa","type":"private","url":"https://www.absa.africa/absaafrica/careers/","active":true,"crawl_strategy":"html","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Banking careers, graduate and internship programmes."},
  {"source_id":"nedbank","name":"Nedbank","type":"private","url":"https://www.nedbank.co.za/content/nedbank/desktop/gt/en/careers.html","active":true,"crawl_strategy":"html","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Banking careers, internships and bursaries."},
  {"source_id":"mtn","name":"MTN Group","type":"private","url":"https://group.mtn.com/careers/","active":true,"crawl_strategy":"html","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Telecommunications careers and graduate programmes."},
  {"source_id":"vodacom","name":"Vodacom","type":"private","url":"https://www.vodacom.com/careers.php","active":true,"crawl_strategy":"html","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Telecommunications careers, graduate and youth programmes."},
  {"source_id":"sasol","name":"Sasol","type":"private","url":"https://www.sasol.com/careers","active":true,"crawl_strategy":"html","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Energy and chemicals careers, apprenticeships and bursaries."},
  {"source_id":"shoprite","name":"Shoprite Group","type":"private","url":"https://www.shopriteholdings.co.za/careers.html","active":true,"crawl_strategy":"html","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Retail careers, bursaries and graduate programmes."},
  {"source_id":"pick-n-pay","name":"Pick n Pay","type":"private","url":"https://www.pnp.co.za/careers","active":true,"crawl_strategy":"html","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Retail careers and graduate opportunities."},
  {"source_id":"woolworths","name":"Woolworths South Africa","type":"private","url":"https://www.woolworths.co.za/corporate/careers","active":true,"crawl_strategy":"html","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Retail careers and early-career programmes."},
  {"source_id":"city-of-johannesburg","name":"City of Johannesburg","type":"municipality","url":"https://www.joburg.org.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Metropolitan municipality vacancies, internships and learnerships."},
  {"source_id":"city-of-cape-town","name":"City of Cape Town","type":"municipality","url":"https://www.capetown.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Metropolitan municipality jobs and skills programmes."},
  {"source_id":"ethekwini","name":"eThekwini Municipality","type":"municipality","url":"https://www.durban.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Durban metro vacancies, internships and bursaries."},
  {"source_id":"tshwane","name":"City of Tshwane","type":"municipality","url":"https://www.tshwane.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Pretoria metro vacancies and youth opportunities."},
  {"source_id":"ekurhuleni","name":"City of Ekurhuleni","type":"municipality","url":"https://www.ekurhuleni.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Metro vacancies, internships and learnerships."},
  {"source_id":"nelson-mandela-bay","name":"Nelson Mandela Bay Municipality","type":"municipality","url":"https://www.nelsonmandelabay.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Metro vacancies and skills development notices."},
  {"source_id":"mangaung","name":"Mangaung Metropolitan Municipality","type":"municipality","url":"https://www.mangaung.co.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Metro vacancies, internships and public notices."},
  {"source_id":"buffalo-city","name":"Buffalo City Metropolitan Municipality","type":"municipality","url":"https://www.buffalocity.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Metro vacancies and youth-development notices."},
  {"source_id":"msunduzi","name":"Msunduzi Municipality","type":"municipality","url":"https://www.msunduzi.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":3,"last_checked":null,"last_success":null,"notes":"Large local municipality vacancies and internships."},
  {"source_id":"stellenbosch-municipality","name":"Stellenbosch Municipality","type":"municipality","url":"https://stellenbosch.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":3,"last_checked":null,"last_success":null,"notes":"Municipal vacancies and public opportunity notices."},
  {"source_id":"western-cape-government","name":"Western Cape Government","type":"government","url":"https://www.westerncape.gov.za/jobs","active":true,"crawl_strategy":"html","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Provincial government vacancies and internships."},
  {"source_id":"gauteng-government","name":"Gauteng Provincial Government","type":"government","url":"https://www.gauteng.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Provincial vacancies, internships and skills programmes."},
  {"source_id":"kzn-government","name":"KwaZulu-Natal Provincial Government","type":"government","url":"https://www.kznonline.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Provincial government vacancies and programme notices."},
  {"source_id":"csir","name":"Council for Scientific and Industrial Research (CSIR)","type":"government","url":"https://www.csir.co.za/careers","active":true,"crawl_strategy":"html","frequency":"weekly","priority":2,"last_checked":null,"last_success":null,"notes":"Research careers, studentships and internships."},
  {"source_id":"nsfas","name":"National Student Financial Aid Scheme (NSFAS)","type":"government","url":"https://www.nsfas.org.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Student funding announcements and operational opportunities."},
  {"source_id":"dhet","name":"Department of Higher Education and Training","type":"government","url":"https://www.dhet.gov.za/","active":true,"crawl_strategy":"hybrid","frequency":"weekly","priority":1,"last_checked":null,"last_success":null,"notes":"Post-school education opportunities, bursaries and skills programmes."}
]
````

## File: docs/architecture/data.md
````markdown
# Data Architecture

## Canonical Source
`data/` directory contains the single source of truth.

## Files
| File | Purpose | Schema | Records |
|---|---|---|---|
| `opportunities.json` | All opportunity listings | See `docs/schema-draft.json` | ~50+ |
| `guides.json` | Educational guide articles | Title, content, tags, date | ~10+ |
| `categories.json` | Category metadata | Name, slug, description, icon | ~15 |
| `provinces.json` | Province list | Name, slug, code | 9 |

## Data Flow
1. Content added to `data/*.json`
2. Run `node validate-content.js` to validate
3. Run `node sitemap-generator.js` to update sitemap
4. Run `node scripts/check-links.js` to verify URLs
5. Commit and push
6. GitHub Pages serves updated JSON
7. `index.html` fetches fresh data on load

## Validation Pipeline
```bash
npm run validate   # validate-content.js
npm run links      # check-links.js
npm run sitemap    # sitemap-generator.js
npm run check      # full release check
```

## Schema Contract (Permanent — Never Change)

Add to all data files:
```json
{
  "_schema_version": "1.0",
  "_last_updated": "2026-07-04",
  "opportunities": [...]
}
```

### Field Rules
- All fields: snake_case, immutable names
- Dates: `YYYY-MM-DD`
- Booleans: `true`/`false` (not strings)
- Arrays: always valid JSON arrays
- URLs: must include protocol (`https://`)
- No trailing commas
- 2-space indentation

### Required Fields (Every Opportunity)
- `id` (string)
- `slug` (string, unique)
- `title` (string)
- `category` (string)
- `sector` (string)
- `province` (string)
- `closing_date` (string, YYYY-MM-DD)
- `application_url` (string, URL)
- `source_name` (string)
- `source_url` (string, URL)
- `verified` (boolean)
- `expired` (boolean)

## Temporary Limitations
- No concurrent editing (merge conflicts)
- No server-side validation at write time
- No user-generated content
- File size will grow (multi-MB eventually)
- No analytics on data access

## Migration Path
- Phase 3: PostgreSQL database + REST API
- Phase 4: Headless CMS (Directus/Strapi)
- Phase 5: AI-assisted content pipeline

## Prohibited
- Never store data in `index.html`
- Never bypass validation before pushing
- Never delete expired opportunities — mark `expired: true`
- Never rename fields without backward compatibility plan
````

## File: docs/architecture/frontend.md
````markdown
# Frontend Architecture

## Single File SPA
The current frontend is a temporary launch-phase monolith. The entire website is contained in `index.html` (~94KB). It is a self-contained Single Page Application using:
- Vanilla JavaScript (no frameworks)
- CSS embedded in `<style>` tags
- HTML templates rendered via JS
- Client-side routing via hash fragments (`#/opportunity/slug`)

This single-file approach is intentional for Phase 1 speed and GitHub Pages compatibility, but it is not the target architecture. It should be treated as scaffolding that will be extracted into reusable components, pages, hooks, styles, and routing utilities during Phase 2.

## Entry Point
- **File:** `index.html`
- **Load sequence:** HTML → CSS → JS → fetch `/data/opportunities.json` → render
- **Hosting contract:** GitHub Pages serves the root `index.html` file directly.
- **Data contract:** The frontend consumes static JSON from `/data/`, especially `/data/opportunities.json`.

## Key Sections in index.html

The monolith should be annotated with component boundary comments before extraction work begins. These markers are documentation breadcrumbs for future maintainers and migration tooling.

### Component Boundary Marker Format
Use this exact comment pattern around each logical UI section:

```html
<!-- COMPONENT: ComponentName -->
<!-- MIGRATION: Will become src/components/ComponentName.jsx -->
<!-- DEPENDENCIES: dependency names or None -->
<!-- PROPS: propName (type), anotherProp (type) -->
<section-or-div>...</section-or-div>
<!-- END COMPONENT: ComponentName -->
```

For sections that do not accept props, omit the `PROPS` line or write `<!-- PROPS: None -->`. For route-level sections, use the future `src/pages/` location in the migration comment.

### Header
```html
<!-- COMPONENT: Header -->
<!-- MIGRATION: Will become src/components/Header.jsx -->
<!-- DEPENDENCIES: None -->
<header>...</header>
<!-- END COMPONENT: Header -->
```

### OpportunityCard
```html
<!-- COMPONENT: OpportunityCard -->
<!-- MIGRATION: Will become src/components/OpportunityCard.jsx -->
<!-- PROPS: opportunity (object) -->
<div class="opportunity-card">...</div>
<!-- END COMPONENT: OpportunityCard -->
```

### SearchBar
```html
<!-- COMPONENT: SearchBar -->
<!-- MIGRATION: Will become src/components/SearchBar.jsx -->
<!-- DEPENDENCIES: search state, filter state -->
<!-- PROPS: query (string), onSearch (function) -->
<form class="search-bar">...</form>
<!-- END COMPONENT: SearchBar -->
```

### FilterPanel
```html
<!-- COMPONENT: FilterPanel -->
<!-- MIGRATION: Will become src/components/FilterPanel.jsx -->
<!-- DEPENDENCIES: categories data, provinces data, filter state -->
<!-- PROPS: filters (object), categories (array), provinces (array), onFilterChange (function) -->
<aside class="filter-panel">...</aside>
<!-- END COMPONENT: FilterPanel -->
```

### Footer
```html
<!-- COMPONENT: Footer -->
<!-- MIGRATION: Will become src/components/Footer.jsx -->
<!-- DEPENDENCIES: None -->
<footer>...</footer>
<!-- END COMPONENT: Footer -->
```

### Route-Level Page Boundaries
Route render blocks should use the same marker format, but point to `src/pages/`:

```html
<!-- COMPONENT: HomePage -->
<!-- MIGRATION: Will become src/pages/HomePage.jsx -->
<!-- DEPENDENCIES: OpportunityCard, SearchBar, FilterPanel -->
<!-- PROPS: opportunities (array), filters (object) -->
<main>...</main>
<!-- END COMPONENT: HomePage -->
```

Known page targets include:
- `HomePage.jsx` for the default opportunity listing.
- `OpportunityDetailPage.jsx` for individual opportunity detail routes.
- `CategoryPage.jsx` for category-filtered listings.
- `ProvincePage.jsx` for province-filtered listings.

## Routing
- Hash-based routing is used for GitHub Pages compatibility.
- Current route patterns:
  - `/#/`
  - `/#/opportunity/{slug}`
  - `/#/category/{slug}`
  - `/#/province/{slug}`
- `404.html` handles the GitHub Pages SPA fallback so deep links can resolve back into the client-side application.

The hash router is temporary. It should be replaced with History API routing during the component/build migration and then with SSR-compatible routing during Phase 4.

## State Management
- Pure JavaScript objects are used for UI and data state.
- There is no external state management library.
- Opportunities are loaded from `/data/opportunities.json` and held client-side.
- Search and filter state live in browser memory during the session.
- Bookmarks are stored in `localStorage` under the key `careerhub_bookmarks`.

The current state model is acceptable for launch, but it couples rendering, routing, data loading, and persistence inside the monolith. During extraction, state responsibilities should move into focused hooks such as `useOpportunities`, `useSearch`, and `useBookmarks`.

## Dependencies
- None. Zero external JavaScript libraries.
- No bundler.
- No transpiler.
- No framework runtime.
- No package-managed frontend build pipeline.

This dependency-free approach keeps Phase 1 deployment simple, but it also prevents component isolation, typed interfaces, tree-shaking, and modern developer tooling.

## Temporary Limitations
The monolithic frontend is explicitly temporary and should not be expanded indefinitely. Known limitations include:
- No component reusability (copy-paste HTML/JS)
- No separation of concerns (HTML mixed with CSS mixed with JS)
- No testing isolation (can't unit test functions in 94KB file)
- No hot reloading (must edit one giant file)
- No lazy loading (entire app loads at once)
- No type safety for opportunity objects, route params, or UI state
- No route-level code splitting
- No dynamic per-route metadata for SEO
- No independent visual testing for cards, filters, headers, or page layouts

## Migration Path

### Phase 2: Component Extraction (Month 3–4)
Extract the monolith into a build-backed frontend structure. The initial target may use React, Vue, or Svelte through Vite, but the planned component names and boundaries are:

```text
src/
├── components/
│   ├── Header.jsx
│   ├── OpportunityCard.jsx
│   ├── SearchBar.jsx
│   ├── FilterPanel.jsx
│   └── Footer.jsx
├── pages/
│   ├── HomePage.jsx
│   ├── OpportunityDetailPage.jsx
│   ├── CategoryPage.jsx
│   └── ProvincePage.jsx
├── hooks/
│   ├── useOpportunities.js
│   ├── useSearch.js
│   └── useBookmarks.js
├── styles/
│   └── variables.css
└── utils/
    └── router.js
```

Phase 2 extraction steps:
1. Add component boundary comments to `index.html` before moving code.
2. Extract static layout sections first (`Header`, `Footer`).
3. Extract repeated UI units (`OpportunityCard`).
4. Extract interactive controls (`SearchBar`, `FilterPanel`).
5. Extract route-level page render functions into `pages/`.
6. Move data loading into `useOpportunities`.
7. Move search/filter logic into `useSearch`.
8. Move bookmark persistence into `useBookmarks`.
9. Move hash parsing and route generation into `utils/router.js`.
10. Add unit tests around hooks and utilities before changing data contracts.

### Phase 3: Platform Integration (Month 5–6)
When the backend and database arrive, keep the frontend data contract stable while replacing static JSON reads with API calls:
- Preserve opportunity field names for backward compatibility.
- Keep bookmark UX unchanged while adding authenticated cloud sync.
- Add loading, empty, and error states around API requests.
- Introduce environment-specific API base URLs.

### Phase 4: SSR (Month 7–8)
- Migrate to Next.js or Nuxt.
- Server-side render opportunity pages for SEO.
- Generate dynamic meta tags per opportunity.
- Replace hash routes with clean canonical URLs.
- Add structured data per detail page.
- Preserve GitHub Pages-era URLs with redirects where possible.

## Architecture Rules Until Extraction
- Do not rename `index.html` while GitHub Pages is the host.
- Do not introduce a build step without executing the Phase 2 migration plan.
- Do not add external frontend dependencies casually; the current dependency contract is zero external JS libraries.
- Do not rename JSON fields consumed by the frontend without a backward compatibility plan.
- Keep the Search → Filter → View → Bookmark → Apply user flow intact during every migration phase.
````

## File: docs/architecture/overview.md
````markdown
# Architecture Overview — CareerHub SA

## Purpose
South Africa's career opportunity intelligence platform for learnerships, bursaries, internships, and apprenticeships.

## Stack
- **Frontend:** Vanilla HTML5 / CSS3 / JavaScript (ES6+)
- **Hosting:** GitHub Pages (static site)
- **Data:** JSON files served statically from `/data/`
- **Build:** Zero build step — no bundler, no transpiler
- **Domain:** opportunitiesza.co.za (custom domain via CNAME)

## Temporary Scaffolding (Phase 1 Only)

These exist now for rapid launch but will be replaced as the project scales.

| Feature | Current | Target | Migration Phase | Why Temporary |
|---|---|---|---|---|
| Hosting | GitHub Pages | Vercel/Netlify → VPS | Phase 3 | No server-side processing, rate limits |
| Frontend | Single `index.html` (~94KB) | Vite + React/Vue/Svelte components | Phase 2 | Unmaintainable monolith, no testing isolation |
| Data | Static JSON files | PostgreSQL + REST API | Phase 3 | No concurrent editing, file size limits, no user-generated content |
| Build | Zero build step | Vite build pipeline | Phase 2 | No component reusability, no TypeScript, no tree-shaking |
| Search | Client-side only | Algolia/Typesense → Elasticsearch | Phase 4 | Loads entire dataset, no fuzzy matching, no analytics |
| Bookmarks | localStorage | Cloud-synced user accounts | Phase 3 | Device-specific, no sync, cleared on browser reset |
| Routing | Hash-based (`/#/`) | History API + SSR | Phase 4 | Bad for SEO, ugly URLs, no dynamic OG tags per page |
| Auth | None | JWT → OAuth | Phase 3 | Can't personalize, can't sync bookmarks, no admin roles |
| Crawler | Manual/undocumented | Scheduled AI pipeline | Phase 5 | No automation, no deduplication, no NLP parsing |
| Publishing | Manual git push | Auto-publish after review | Phase 5 | High error rate, no workflow (draft → review → publish) |
| Validation | Manual script runs | CI/CD automated | Phase 3 | Easy to forget, no enforcement |
| SEO | Basic OG tags | Full structured data + SSR | Phase 4 | No JSON-LD, no dynamic meta tags per opportunity |

## Permanent Contracts (Never Change)

These survive all architecture migrations:

- **Data schema:** Field names, types, structure — the API contract
- **User flow:** Search → Filter → View → Bookmark → Apply
- **SEO requirements:** sitemap.xml, OG tags, structured data, canonical URLs
- **Source governance:** Every opportunity must have verified official source
- **Mobile-first responsive design:** All UI must work on 320px+
- **Content validation rules:** Required fields, date formats, URL protocols

## Public Interfaces

| Interface | Location | Consumers | Stability |
|---|---|---|---|
| Opportunities API | `data/opportunities.json` | Frontend, sitemap generator | Permanent schema |
| Guides API | `data/guides.json` | Frontend | Permanent schema |
| Categories API | `data/categories.json` | Frontend | Permanent schema |
| Provinces API | `data/provinces.json` | Frontend | Permanent schema |
| Sitemap | `sitemap.xml` | Search engines | Auto-generated |

## Prohibited Modifications

- **NEVER** rename `index.html` — GitHub Pages requires it at root
- **NEVER** add a build step (Webpack, Vite, etc.) without Phase 2 migration plan
- **NEVER** commit `node_modules/`
- **NEVER** modify `CNAME` without DNS preparation
- **NEVER** delete data records — mark `expired: true` (retains SEO value)
- **NEVER** rename JSON field names without backward compatibility plan
- **NEVER** break the Search → Filter → View → Bookmark → Apply user flow

## Repository Boundaries

| Task Type | Allowed Directories | Forbidden Directories |
|---|---|---|
| UI Feature | `index.html` | `data/`, `scripts/`, `crawler/` |
| Content | `data/*.json` | `index.html`, `scripts/` |
| Script | `scripts/` | `index.html`, `data/` (read-only) |
| SEO | `index.html`, `sitemap.xml`, `robots.txt` | `data/`, `scripts/` |
| Documentation | `docs/` | `index.html`, `data/`, `scripts/` |

## Migration Roadmap

| Phase | Timeline | Focus | Key Deliverables |
|---|---|---|---|
| Phase 1: Launch | Now | Static scaffolding | GitHub Pages, JSON data, manual workflows |
| Phase 2: Structure | Month 3–4 | Build step + components | Vite, component extraction, TypeScript |
| Phase 3: Platform | Month 5–6 | Backend + database | Vercel/Netlify, PostgreSQL, REST API, JWT auth |
| Phase 4: Scale | Month 7–8 | SSR + search engine | Next.js/Nuxt, Algolia/Typesense, headless CMS |
| Phase 5: Intelligence | Month 9–12 | AI automation | AI crawler, NLP parser, review queue, auto-publish, ML scoring |
````

## File: docs/architecture/Repository-Workflow-Optimization-Framework.md
````markdown
# Repository Workflow Optimization Framework
## Codex Context / Token Usage Reduction Framework

Based on the conversations in this project, there are two major themes that were developed in depth:

1. Repository Workflow Optimization Framework
2. Codex Context / Token Usage Reduction Framework

These discussions evolved into a methodology for building large repositories with AI while minimizing unnecessary context loading, reducing prompt sizes, increasing task completion rates, and making f[...]

---

## PART I — Repository Workflow Optimization Framework

The repository workflow was designed around a modular AI-first development process.

Instead of treating the repository as one large project, it is divided into independent systems.

The goal is:

- minimize AI context
- isolate failures
- improve maintainability
- allow parallel development
- reduce hallucinations
- improve debugging
- improve testing

### 1 Repository as Independent Systems

Rather than:
```
Repository
    ↓
Everything connected
```

the repository becomes:
```
Repository
├── Frontend
├── Backend
├── Data Sources
├── Scrapers
├── API
├── Validation
├── Scheduler
├── Parser
├── Documentation
├── Tests
└── Deployment
```

Each system owns its own:
- documentation
- architecture
- tests
- configuration
- AI context

Meaning Codex only loads the section being modified.

### 2 Layered Architecture

Development is separated into layers.

Example:
```
UI
  ↓
Business Logic
  ↓
Services
  ↓
Data Layer
  ↓
Storage
  ↓
Configuration
```

Each layer communicates through interfaces rather than directly.

**Benefits:**
- fewer dependencies
- easier debugging
- smaller AI context
- isolated testing

### 3 Feature Isolation

Every feature is treated as its own module.

Instead of:
```
Everything imports everything
```

Use:
```
Feature A
├── docs
├── tests
├── logic
└── config

Feature B
├── docs
├── tests
├── logic
└── config
```

AI only needs Feature B to modify Feature B.

### 4 Task-Based Development

Large prompts become hundreds of small tasks.

Example: Instead of "Build scraping system", split into:
- Task 1: Create source loader
- Task 2: Validate sources
- Task 3: Create parser interface
- Task 4: Implement PDF parser
- Task 5: Unit tests

Each task loads dramatically fewer files.

### 5 Strict Repository Boundaries

Every task clearly defines:
- Allowed folders
- Forbidden folders
- Files to modify
- Files to create
- Expected outputs
- Success criteria
- Rollback strategy

This prevents Codex from exploring the entire repository.

### 6 Documentation-Driven Development

Documentation becomes the primary source of truth.

Examples:
```
docs/
├── architecture
├── modules
├── rules
├── context
├── workflow
├── templates
├── contracts
└── interfaces
```

Instead of loading 300 source files, AI loads a few documentation files.

### 7 Phase-Based Development

Development progresses in phases:
- Phase 0: Repository structure
- Phase 1: Infrastructure
- Phase 2: Core engine
- Phase 3: Scrapers
- Phase 4: API
- Phase 5: UI
- Phase 6: Testing
- Phase 7: Deployment

Each phase has independent documentation.

---

## PART II — Codex Token Usage Reduction Framework

This became one of the major focuses of the project.

**Goal:** Reduce context size while increasing output quality.

### Core Principle

Never make Codex understand the whole repository.

Instead: Make Codex understand one task.

### 1 Context Isolation

Every task receives only:
- Objective
- Files
- Dependencies
- Expected outputs
- Success criteria
- Nothing else.

Instead of asking to "Understand entire repository", provide "Modify: scripts/parser.js Only."

Huge token reduction.

### 2 Context Hierarchy

The framework defines multiple context levels:
```
Repository Context
  ↓
Module Context
  ↓
Feature Context
  ↓
Task Context
  ↓
File Context
  ↓
Function Context
```

Codex should only receive the lowest necessary level.

### 3 Context Files

Dedicated documentation files were proposed:

```
docs/context/
├── repository.md
├── frontend.md
├── backend.md
├── parser.md
├── api.md
├── scraper.md
├── scheduler.md
└── validation.md
```

Codex reads only one context file.

### 4 AI Rules

Create `docs/rules/` containing:
- Naming conventions
- Folder rules
- Coding standards
- Architecture rules
- Import rules
- Testing rules
- Documentation rules

Instead of explaining rules every prompt.

### 5 Prompt Templates

Instead of rewriting prompts, store templates:

```
docs/templates/
├── bug-fix.md
├── feature.md
├── refactor.md
├── documentation.md
├── testing.md
├── review.md
└── optimization.md
```

### 6 Scope Limitation

Every task contains:
- **Scope**: What can be modified
- **Out of scope**: What cannot be touched
- **Forbidden actions**: What is prohibited

Example:
```
ONLY modify:
- scripts/parser.js

Do NOT modify:
- data
- UI
- tests
- package.json
```

Codex explores dramatically fewer files.

### 7 Explicit Constraints

Tasks define:
- May create
- May edit
- Must not edit
- Allowed dependencies
- Forbidden dependencies
- Expected filenames
- Expected functions
- Maximum files modified

This prevents unnecessary exploration.

### 8 Small Independent Tasks

Large "Implement scraper" becomes:
- Task A: Create interface
- Task B: Validate URLs
- Task C: Normalize output
- Task D: Error handling
- Task E: Tests
- Task F: Documentation

Each prompt becomes tiny.

### 9 Task Chaining

Rather than one huge prompt, use sequential tasks:
```
Task 1
  ↓
Task 2
  ↓
Task 3
  ↓
Task 4
```

Each task references outputs from previous tasks instead of repeating full context.

### 10 Repository Memory

Persistent documentation replaces repeated explanations:
- architecture.md
- coding-rules.md
- interfaces.md
- contracts.md

Codex loads these instead of rediscovering the repository.

### 11 Interface Contracts

Modules communicate through documented interfaces:
```
Input
  ↓
Validation
  ↓
Parser
  ↓
Normalizer
  ↓
Storage
```

Each interface is documented. Codex doesn't inspect every module.

### 12 Stable APIs

Document:
- Inputs
- Outputs
- Types
- Errors
- Examples

No source-code exploration required.

### 13 AI Context Compression

Instead of loading 100 files, summarize into:
- One architecture file
- One interface file
- One workflow file
- One module file

This dramatically reduces prompt size.

### 14 Documentation Before Code

Before implementation, create:
- Architecture
- Contracts
- Interfaces
- Workflow
- Dependencies

Then implementation. AI reads documentation instead of discovering code.

### 15 Standard Task Structure

Every Codex prompt follows a fixed structure:
1. Objective
2. Scope
3. Files allowed
4. Files forbidden
5. Expected output
6. Constraints
7. Acceptance criteria

This standardization reduces ambiguity and unnecessary context.

### 16 Dependency Mapping

Document module dependencies explicitly. Instead of allowing AI to infer relationships, provide a dependency graph showing which modules interact and which are isolated. This reduces exploratory code reading.

### 17 Incremental Repository Indexing

Maintain lightweight index files that summarize repository structure, major modules, and entry points. AI consults these indexes first before any code, avoiding repeated directory traversal.

### 18 Architecture Decision Records (ADRs)

Capture important architectural decisions in dedicated documents. Future tasks reference ADRs instead of rediscovering why design choices were made, preventing repeated analysis.

### 19 Success Criteria-Driven Prompts

Every task specifies measurable completion criteria such as:
- Required files created or modified
- Tests passing
- Documentation updated
- No unrelated files changed

This limits over-implementation.

### 20 Failure and Rollback Guidance

Include expected failure handling and rollback instructions within task prompts so Codex can avoid cascading changes when a step cannot be completed.

### 21 Documentation Directory Structure

A dedicated documentation hierarchy was proposed to support context compression:

```
docs/
├── architecture/
├── context/
├── rules/
├── templates/
├── workflow/
├── interfaces/
├── contracts/
├── modules/
├── phases/
├── testing/
├── deployment/
└── decisions/
```

Each directory serves a distinct purpose, allowing AI to load only the documentation relevant to the current task.

### 22 Prompt Design Principles

The discussions consistently emphasized that effective prompts should:
- Be single-objective
- Have explicit scope boundaries
- Define allowed and forbidden files
- Specify expected outputs and acceptance criteria
- Avoid repeating repository-wide context
- Reference existing documentation instead of embedding large explanations
- Be reusable through standardized templates

### 23 Expected Benefits

Implementing the combined Repository Workflow Optimization Framework and Codex Token Usage Reduction Framework is expected to provide:
- Significant reductions in prompt and context token consumption
- Lower AI operating costs
- Faster task completion
- Higher completion accuracy
- Reduced hallucinations
- Fewer unintended repository modifications
- Improved maintainability and onboarding
- Better modularity and separation of concerns
- Easier testing and debugging
- Greater consistency across AI-assisted development sessions
- Reusable documentation and prompt assets that compound efficiency over time

---

Together, these discussions define an AI-first repository engineering methodology: structure the repository around modular components, capture architecture and rules in persistent documentation, and constrain AI tasks to minimal, focused scopes.
````

## File: docs/context/data-context.md
````markdown
# Data Context

- Scope: JSON data files only.
- Purpose:
  - Store structured records used by the application.
  - Preserve stable IDs and record shape.
  - Support focused record-level edits.
- Default context:
  - Load only the target JSON file when required.
  - Load only the target record by ID when possible.
  - Do not load full datasets by default.
- Editing principle:
  - Prefer one-record changes.
  - Preserve unrelated records and ordering unless instructed.
  - Keep schema-compatible fields unchanged unless targeted.
- Cross-file updates:
  - Require explicit instruction.
  - Name every affected file and ID.
  - Apply only the requested linked changes.
- Related rules:
  - `docs/rules/data-access-rules.md`
  - `docs/rules/context-budget-rules.md`
````

## File: docs/context/scripts-context.md
````markdown
# Scripts Context

- Scope: repository scripts only.
- Responsibilities:
  - Automate data checks, transformations, imports, exports, or maintenance.
  - Keep repeatable operations outside manual workflows.
- Task scope:
  - Use one script per task.
  - Use one function per task when editing existing scripts.
  - Avoid unrelated script review.
- Context loading:
  - Load the primary script only.
  - Load at most two supporting files when required.
  - Load only dependencies referenced by the target function.
- Dependency principle:
  - Reuse existing imports.
  - Do not add dependencies unless explicitly requested.
  - Avoid broad module loading for local function edits.
- Related rules:
  - `docs/rules/context-budget-rules.md`
  - `docs/templates/rulesets.md#script_function_edit_v1`
````

## File: docs/context/ui-context.md
````markdown
# UI Context

- Scope: `index.html` only.
- Responsibilities:
  - Define application structure, UI sections, inline behavior, and static references.
  - Preserve current user-facing behavior unless explicitly changed.
- Reasoning scope:
  - Work at section level.
  - Identify the target section before editing.
  - Avoid full SPA reinterpretation.
- Context loading:
  - Load only the target section by default.
  - Load minimal adjacent markup, styles, or script needed for the section.
  - Do not scan unrelated sections unless requested.
- Supporting context:
  - Use named IDs, classes, handlers, or data attributes as anchors.
  - Include data or script context only when directly required.
- Related rules:
  - `docs/rules/context-budget-rules.md`
  - `docs/templates/rulesets.md#index_html_section_edit_v1`
````

## File: docs/modules/module-index.md
````markdown
# Module Index

## Module 01 — Authentication
- **Status:** Not implemented
- **Entry Point:** N/A
- **Dependencies:** None
- **Priority:** Low
- **Phase:** 3

## Module 02 — Homepage
- **Status:** Implemented in `index.html`
- **Entry Point:** `index.html` (route: `/`)
- **Dependencies:** Opportunity model, Category data, Province data
- **Priority:** High (maintain)
- **Phase:** 1

## Module 03 — Navigation
- **Status:** Implemented in `index.html`
- **Entry Point:** `index.html` (header/nav section)
- **Dependencies:** None
- **Priority:** High (enhance mobile menu)
- **Phase:** 1

## Module 04 — Search
- **Status:** Implemented in `index.html`
- **Entry Point:** `index.html` (search section)
- **Dependencies:** Opportunity model
- **Priority:** High (add filters)
- **Phase:** 1

## Module 05 — Opportunity Details
- **Status:** Implemented in `index.html`
- **Entry Point:** `index.html` (route: `/#/opportunity/{slug}`)
- **Dependencies:** Opportunity model, Bookmark module
- **Priority:** High (add related opportunities)
- **Phase:** 1

## Module 06 — Bookmarks
- **Status:** Implemented in `index.html`
- **Entry Point:** `index.html` (localStorage + bookmark buttons)
- **Dependencies:** None
- **Priority:** Medium (add export)
- **Phase:** 1

## Module 07 — SETA Directory
- **Status:** Partial (categories exist)
- **Entry Point:** `index.html` (category grid)
- **Dependencies:** Category data
- **Priority:** Medium (dedicated page)
- **Phase:** 2

## Module 08 — Province Pages
- **Status:** Partial (province filter exists)
- **Entry Point:** `index.html` (province grid + filter)
- **Dependencies:** Province data, Opportunity model
- **Priority:** Medium (dedicated pages)
- **Phase:** 2

## Module 09 — Crawler
- **Status:** Exists but undocumented
- **Entry Point:** `crawler/` directory
- **Dependencies:** Source registry, Data schema
- **Priority:** High (document and automate)
- **Phase:** 5

## Module 10 — Parser
- **Status:** Not implemented
- **Entry Point:** N/A
- **Dependencies:** Crawler output
- **Priority:** Medium
- **Phase:** 5

## Module 11 — Review Queue
- **Status:** Not implemented
- **Entry Point:** N/A
- **Dependencies:** Parser output
- **Priority:** Low
- **Phase:** 5

## Module 12 — Publishing
- **Status:** Manual (edit JSON, validate, push)
- **Entry Point:** `data/*.json`
- **Dependencies:** Validation scripts
- **Priority:** Medium (automate)
- **Phase:** 5

## Module 13 — Admin Dashboard
- **Status:** Not implemented
- **Entry Point:** N/A
- **Dependencies:** Authentication, Publishing
- **Priority:** Low
- **Phase:** 4

## Module 14 — SEO
- **Status:** Partial (sitemap, robots, OG tags)
- **Entry Point:** `index.html`, `sitemap.xml`, `robots.txt`
- **Dependencies:** Content data
- **Priority:** High (enhance structured data)
- **Phase:** 1

## Module 15 — Deployment
- **Status:** Implemented (GitHub Pages)
- **Entry Point:** `deploy.sh`, GitHub Actions
- **Dependencies:** None
- **Priority:** Medium (add CI validation)
- **Phase:** 1
````

## File: docs/phases/backlog.md
````markdown
# Development Backlog

## Phase A — Documentation Foundation (Weeks 1–2)
- [ ] TASK-001: Create docs/architecture/overview.md
- [ ] TASK-002: Create docs/architecture/frontend.md
- [ ] TASK-003: Create docs/architecture/data.md
- [ ] TASK-004: Create docs/architecture/scripts.md
- [ ] TASK-005: Create docs/architecture/deployment.md
- [ ] TASK-006: Create docs/modules/module-index.md
- [ ] TASK-007: Create docs/modules/homepage.md
- [ ] TASK-008: Create docs/modules/search.md
- [ ] TASK-009: Create docs/modules/opportunity-detail.md
- [ ] TASK-010: Create docs/modules/bookmarks.md
- [ ] TASK-011: Create docs/modules/navigation.md
- [ ] TASK-012: Create docs/modules/seo.md
- [ ] TASK-013: Create docs/modules/crawler.md
- [ ] TASK-014: Create docs/modules/parser.md
- [ ] TASK-015: Create docs/modules/publishing.md
- [ ] TASK-016: Create docs/modules/seta-directory.md
- [ ] TASK-017: Create docs/modules/province-pages.md
- [ ] TASK-018: Create docs/modules/admin-dashboard.md
- [ ] TASK-019: Create docs/modules/authentication.md
- [ ] TASK-020: Create docs/modules/deployment.md
- [ ] TASK-021: Create docs/phases/dependency-map.md
- [ ] TASK-022: Create docs/phases/task-examples.md
- [ ] TASK-023: Create docs/rules/boundaries.md
- [ ] TASK-024: Create docs/rules/coding.md
- [ ] TASK-025: Create docs/rules/naming.md
- [ ] TASK-026: Create docs/rules/testing.md
- [ ] TASK-027: Create docs/rules/commits.md
- [ ] TASK-028: Create docs/context/repository-summary.md
- [ ] TASK-029: Create docs/context/frontend-context.md
- [ ] TASK-030: Create docs/context/data-context.md
- [ ] TASK-031: Create docs/context/scripts-context.md
- [ ] TASK-032: Create docs/context/crawler-context.md
- [ ] TASK-033: Create docs/context/seo-context.md
- [ ] TASK-034: Create docs/templates/codex-task-template.md
- [ ] TASK-035: Create docs/templates/feature-implementation.md
- [ ] TASK-036: Create docs/templates/bug-fix.md
- [ ] TASK-037: Create docs/templates/refactoring.md
- [ ] TASK-038: Create docs/templates/content-update.md
- [ ] TASK-039: Create docs/templates/documentation.md
- [ ] TASK-040: Create docs/workflows/validation.md
- [ ] TASK-041: Create docs/workflows/release.md
- [ ] TASK-042: Create docs/workflows/mobile-orchestration.md
- [ ] TASK-043: Create docs/workflows/ai-development.md

## Phase B — Core Features (Weeks 3–4)
- [ ] TASK-044: Add responsive mobile hamburger menu with submenu dropdown
- [ ] TASK-045: Add "Closing Soon" section to homepage
- [ ] TASK-046: Add "Recently Added" section to homepage
- [ ] TASK-047: Add qualification level filter to search
- [ ] TASK-048: Add SETA filter to search
- [ ] TASK-049: Add pagination to search results
- [ ] TASK-050: Add "Similar Opportunities" to detail page
- [ ] TASK-051: Add "Share to WhatsApp" button on detail page

## Phase C — Abstraction Layer (Week 5)
- [ ] TASK-052: Add CONFIG object to index.html
- [ ] TASK-053: Add Storage abstraction wrapper
- [ ] TASK-054: Add API abstraction wrapper
- [ ] TASK-055: Add Search abstraction wrapper
- [ ] TASK-056: Add Router abstraction wrapper

## Phase D — Automation (Week 6)
- [ ] TASK-057: Add GitHub Actions auto-sitemap on data changes
- [ ] TASK-058: Add GitHub Actions validation on PR
- [ ] TASK-059: Add scheduled link checker (weekly cron)

## Phase E — SEO Enhancement (Week 7)
- [ ] TASK-060: Add JSON-LD structured data to index.html
- [ ] TASK-061: Add Twitter Card meta tags
- [ ] TASK-062: Optimize Core Web Vitals
- [ ] TASK-063: Add canonical URLs

## Phase F — Data Pipeline (Weeks 8–10)
- [ ] TASK-064: Document crawler architecture
- [ ] TASK-065: Add data normalization pipeline
- [ ] TASK-066: Add source health monitoring
- [ ] TASK-067: Create review queue workflow

## Phase G — Scale Preparation (Weeks 11–12)
- [ ] TASK-068: Add bookmark export to JSON
- [ ] TASK-069: Add email reminder for closing dates
- [ ] TASK-070: Create SETA directory page
- [ ] TASK-071: Add search autocomplete
- [ ] TASK-072: Add "No results" suggestions
````

## File: docs/phases/dependency-map.md
````markdown
# Dependency Map

## Frontend Dependencies

```
Homepage
├── Opportunity Model (data structure)
├── Category Data
├── Province Data
└── Search Hook
    └── Opportunity Model

Opportunity Detail
├── Opportunity Model
├── Bookmark Module
└── Share Utilities

Search
├── Opportunity Model
├── Province Data
├── Category Data
└── Search Filters
    ├── Province Filter
    ├── Category Filter
    ├── SETA Filter
    └── Qualification Filter

Bookmarks
└── Storage Abstraction
    └── localStorage (temp) → API (future)
```

## Data Pipeline Dependencies

```
Crawler
├── Source Registry (docs/source-registry.md)
└── robots.txt compliance

Parser
├── Crawler Output
└── Data Schema (docs/schema-draft.json)

Review Queue
├── Parser Output
└── Source Validator (scripts/validators/source-validator.js)

Publishing
├── Review Queue Approval
├── Data Schema Validation
└── Sitemap Regeneration
```

## Implementation Order

1. Data Schema (`docs/schema-draft.json`) — Permanent contract
2. Validation Scripts (`scripts/validate-content.js`)
3. Source Registry (`docs/source-registry.md`)
4. Crawler (documented in `docs/modules/crawler.md`)
5. Parser (after crawler)
6. Review Queue (after parser)
7. Publishing Automation (after review queue)
8. Admin Dashboard (after authentication)

## Rule

Before implementing a module, ALL its dependencies must be documented in `docs/modules/`.
````

## File: docs/rules/boundaries.md
````markdown
# Repository Boundaries

## AI Scope Rules
Every task MUST specify:
- Allowed directories
- Forbidden directories
- Entry files
- Context documents to read first

## Directory Access Matrix

| Task Type | Allowed | Forbidden |
|---|---|---|
| UI Feature | `index.html` | `data/`, `scripts/`, `crawler/` |
| Content | `data/*.json` | `index.html`, `scripts/` |
| Script | `scripts/` | `index.html`, `data/` (read-only) |
| SEO | `index.html`, `sitemap.xml`, `robots.txt` | `data/`, `scripts/` |
| Documentation | `docs/` | `index.html`, `data/`, `scripts/` |

## Temporary Feature Markers
Any code implementing temporary scaffolding MUST include:
:
// TEMPORARY: [feature name]
// TARGET: [permanent replacement]
// MIGRATION: [how to swap]


## Stop Condition
After completing the assigned task and committing, STOP.
````

## File: docs/rules/context-budget-rules.md
````markdown
# Context Budget Rules

- Primary file limit: 1 per task.
- Supporting file limit: 2 per task.
- JSON record scope: 1 record where applicable.
- Directory loading:
  - Do not load full directories.
  - Do not perform broad recursive scans.
  - Request explicit scope before expanding.
- Cross-module scanning:
  - Do not scan across modules unless explicitly requested.
  - Use named files, IDs, functions, or sections as anchors.
- Expansion rule:
  - Expand context only when blocked.
  - State why each added file is required.
````

## File: docs/rules/data-access-rules.md
````markdown
# Data Access Rules

- Use ID-scoped JSON operations.
- Prefer single-record updates.
- Preserve unrelated records.
- Preserve existing field names and schema shape.
- Bulk operations:
  - Require explicit instruction.
  - Require target file list or selection rule.
  - Require expected update behavior.
- Dataset parsing:
  - Avoid full dataset parsing unless necessary.
  - Prefer targeted extraction by ID or key.
  - Stop after the requested record scope is satisfied.
- Cross-file data changes:
  - Require explicit file and ID mapping.
  - Do not infer linked updates without instruction.
````

## File: docs/templates/codex-task-template.md
````markdown
# Codex Task Template

## Task ID
[TASK-XXX from docs/phases/backlog.md]

## Role
You are a [frontend|backend|content|devops] developer for CareerHub SA.

## Objective
[One sentence. Exactly one feature. No "and".]

## Classification
- **Type:** [Feature|Bug|Refactor|Content|Documentation]
- **Scope:** [Temporary scaffolding|Permanent feature]

## Repository Scope

### Allowed Directories
- [List directories AI may read and modify]

### Forbidden Directories
- [List directories AI must NOT touch]

### Entry Files
- [Primary file(s) to modify]

## Context Documents (READ FIRST)
1. [Required doc 1]
2. [Required doc 2]

## Success Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Validation Steps
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Commit Message
```
type(scope): description
```

## Stop Condition
After committing, STOP. Wait for next task.

## Example Task
**Task:** Add a "Closing Soon" section to the homepage.
**Allowed:** `index.html`, `docs/modules/homepage.md`
**Forbidden:** `data/`, `scripts/`, `crawler/`, other docs
**Entry:** `index.html`
**Context:** `docs/modules/homepage.md`, `docs/rules/coding.md`
**Success:** Section appears on homepage, shows opportunities closing within 14 days, is responsive.
**Validation:** Load site, scroll to section, verify cards render, check mobile view.
**Commit:** `feat(homepage): add closing soon section`
````

## File: docs/templates/rulesets.md
````markdown
# Rulesets

## JSON_SINGLE_FILE_EDIT_V1

- Use for one JSON file.
- Target one record by ID when possible.
- Load no full dataset unless required.
- Apply `docs/rules/data-access-rules.md`.
- Apply `docs/rules/context-budget-rules.md`.
- Do not modify unrelated records.

## SCRIPT_FUNCTION_EDIT_V1

- Use for one script function.
- Load one primary script.
- Load only direct supporting files.
- Preserve existing script responsibilities.
- Do not add dependencies unless requested.
- Apply `docs/rules/context-budget-rules.md`.

## INDEX_HTML_SECTION_EDIT_V1

- Use for one `index.html` section.
- Load target section only.
- Load adjacent CSS or JS only when required.
- Preserve unrelated sections and behavior.
- Avoid full SPA reinterpretation.
- Apply `docs/rules/context-budget-rules.md`.
````

## File: docs/blockers.md
````markdown
# Data Automation Blockers

## Current status

There are no known blocking issues for the canonical data automation path.
The repository now uses lowercase `data/` JSON paths consistently for the
runtime app, content validators, sitemap generator, and GitHub Pages workflow.

## Resolved blockers

### Resolved: Data directory casing mismatch

The earlier audit noted that lowercase `/data/...` paths were missing and that
JSON files lived under `/Data/...`. That blocker is no longer current:
canonical files now exist under `data/`, and automation should continue to use
those lowercase paths for Linux and GitHub Pages compatibility.

## Watch list

- Keep generated `sitemap.xml` committed after content changes.
- Run `npm run check` before publishing so JSON validation, link checks, and
  sitemap generation stay aligned.
- Treat external link-check warnings as maintenance prompts; some official
  career portals block automated HEAD/GET requests even when the page is usable
  in a browser.
````

## File: docs/data-audit-report.md
````markdown
# Data Audit Report

## Scope and path note

Requested scan paths were `/data/opportunities.json`, `/data/guides.json`, `/data/categories.json`, and `/data/provinces.json`. On this case-sensitive filesystem those exact lowercase paths are missing; the repository stores the corresponding JSON files under `Data/`, so this report audits only those four corresponding files and does not modify any dataset JSON files.

## Console-style file summaries

### FILE: /data/opportunities.json
- STATUS: missing at /data/opportunities.json; valid audited file at /Data/opportunities.json
- RECORD_COUNT: 12

### FILE: /data/guides.json
- STATUS: missing at /data/guides.json; valid audited file at /Data/guides.json
- RECORD_COUNT: 7

### FILE: /data/categories.json
- STATUS: missing at /data/categories.json; valid audited file at /Data/categories.json
- RECORD_COUNT: 4

### FILE: /data/provinces.json
- STATUS: missing at /data/provinces.json; valid audited file at /Data/provinces.json
- RECORD_COUNT: 9

## Summary of dataset

- `Data/opportunities.json`: 12 records; JSON status: valid.
- `Data/guides.json`: 7 records; JSON status: valid.
- `Data/categories.json`: 4 records; JSON status: valid.
- `Data/provinces.json`: 9 records; JSON status: valid.

## Schema inventory: `Data/opportunities.json`

### Fields found

- application_url
- category
- closing_date
- description
- documents_needed
- eligibility
- expired
- featured
- id
- posted_date
- province
- qualification_level
- sector
- slug
- source_name
- source_url
- stipend
- tags
- title
- updated_date
- verified

### Field types

- application_url → string
- category → string
- closing_date → string
- description → string
- documents_needed → array
- eligibility → array
- expired → boolean
- featured → boolean
- id → string
- posted_date → string
- province → string
- qualification_level → string
- sector → string
- slug → string
- source_name → string
- source_url → string
- stipend → string
- tags → array
- title → string
- updated_date → string
- verified → boolean

### Required fields (present in 95%+ records)

- application_url
- category
- closing_date
- description
- documents_needed
- eligibility
- expired
- featured
- id
- posted_date
- province
- qualification_level
- sector
- slug
- source_name
- source_url
- stipend
- tags
- title
- updated_date
- verified

### Optional fields (present in <95% records)

- None

### Inconsistent presence

- None detected.

### Field usage frequency

- application_url → 100.0%
- category → 100.0%
- closing_date → 100.0%
- description → 100.0%
- documents_needed → 100.0%
- eligibility → 100.0%
- expired → 100.0%
- featured → 100.0%
- id → 100.0%
- posted_date → 100.0%
- province → 100.0%
- qualification_level → 100.0%
- sector → 100.0%
- slug → 100.0%
- source_name → 100.0%
- source_url → 100.0%
- stipend → 100.0%
- tags → 100.0%
- title → 100.0%
- updated_date → 100.0%
- verified → 100.0%

### String value variation analysis

- `application_url` unique values (12): ["https://www.absa.africa/absaafrica/careers", "https://www.capitecbank.co.za/careers", "https://www.eskom.co.za/careers", "https://www.kznhealth.gov.za", "https://www.merseta.org.za", "https://www.mict.org.za/learnerships", "https://www.nsfas.org.za/content/applications.html", "https://www.oldmutual.co.za/careers", "https://www.sasol.com/careers/bursaries", "https://www.shopriteholdings.co.za/careers", "https://www.standardbank.co.za/careers", "https://www.transnet.net/careers"]
- `category` unique values (4): ["apprenticeship", "bursary", "internship", "learnership"]
  - Normalized suggestion: ["apprenticeship", "bursary", "internship", "learnership"]
- `closing_date` unique values (8): ["2026-07-31", "2026-08-15", "2026-08-31", "2026-09-15", "2026-09-30", "2026-10-15", "2026-10-31", "2026-11-30"]
- `description` unique values (12): ["12-month internship placements for graduates in health administration, public health, pharmacy assistance, and environmental health at district offices and provincial hospitals.", "12-month rotational internship across actuarial, finance, IT, and business operations. Structured mentorship with possibility of permanent employment.", "12-month structured programme developing young talent in banking and financial services. Interns rotate across Retail, Corporate, and Digital Banking with structured mentorship.", "12-month YES (Youth Employment Service) placement in banking operations, digital innovation, or financial services. Real work, structured learning, and potential for permanent employment.", "3-year apprenticeship developing qualified artisans in diesel mechanical, electrical, and boilermaking trades. Leads to a nationally recognised trade certificate.", "3-year boilermaker apprenticeship leading to a nationally recognised trade certificate. Covers metal fabrication, welding, and pressure vessel manufacturing.", "Capitec Bank's branch learnership provides NQF Level 4 retail banking qualification through hands-on customer service training across Capitec branches in the Western Cape.", "Eskom's annual learnership programme offers structured workplace learning leading to a nationally recognised NQF qualification. The programme covers electrical, mechanical, and instrumentation disciplines across Eskom's generation, transmission, and distribution divisions.", "Full tuition, accommodation, and monthly living allowance for engineering and science students at any South African public university.", "NQF Level 4 ICT Learnership providing practical experience in IT support, networking, and software fundamentals over 12 months.", "NSFAS provides funding to eligible South African students at public universities and TVET colleges. Covers tuition, accommodation, transport, and a living allowance. No repayment required.", "Retail Learnership across Shoprite, Checkers, and USave providing NQF Level 3 and 4 qualifications in retail operations while working in live store environments."]
- `id` unique values (12): ["001", "002", "003", "004", "005", "006", "007", "008", "009", "010", "011", "012"]
- `posted_date` unique values (11): ["2026-04-01", "2026-04-05", "2026-04-08", "2026-04-10", "2026-04-12", "2026-04-14", "2026-04-15", "2026-04-16", "2026-04-18", "2026-04-20", "2026-04-22"]
- `province` unique values (4): ["Gauteng", "KwaZulu-Natal", "Nationwide", "Western Cape"]
  - Normalized suggestion: ["Gauteng", "KwaZulu-Natal", "Nationwide", "Western Cape"]
- `qualification_level` unique values (2): ["Degree", "Matric"]
- `sector` unique values (10): ["BANKSETA", "CHIETA", "EWSETA", "Government – DHET", "INSETA", "MerSETA", "MICT SETA", "PSETA", "TETA", "W&RSETA"]
- `slug` unique values (12): ["absa-graduate-internship-2026", "capitec-bank-learnership-2026", "eskom-engineering-learnership-2026", "kzn-health-internship-2026", "merseta-boilermaker-apprenticeship-2026", "mict-seta-ict-learnership-2026", "nsfas-bursary-2026", "old-mutual-graduate-internship-2026", "sasol-bursary-engineering-2026", "shoprite-retail-learnership-2026", "standard-bank-yes-internship-2026", "transnet-apprenticeship-2026"]
- `source_name` unique values (12): ["ABSA Group Careers", "Capitec Bank Careers", "Eskom Official Website", "KZN Department of Health", "MerSETA Official Website", "MICT SETA Official", "NSFAS Official Website", "Old Mutual Group Careers", "Sasol Ltd Careers", "Shoprite Holdings Careers", "Standard Bank Careers Portal", "Transnet SOC Ltd"]
- `source_url` unique values (12): ["https://www.absa.africa/absaafrica/careers", "https://www.capitecbank.co.za/careers", "https://www.eskom.co.za/careers", "https://www.kznhealth.gov.za", "https://www.merseta.org.za", "https://www.mict.org.za", "https://www.nsfas.org.za", "https://www.oldmutual.co.za/careers", "https://www.sasol.com/careers", "https://www.shopriteholdings.co.za/careers", "https://www.standardbank.co.za/careers", "https://www.transnet.net/careers"]
- `stipend` unique values (12): ["Full bursary – varies by institution", "Full tuition + R5 000/month", "R3 200/month", "R3 500/month", "R4 000/month", "R4 500/month", "R5 500/month", "R6 000/month", "R6 500/month", "R7 044/month", "R8 500/month", "R9 500/month"]
- `title` unique values (12): ["ABSA Bank Graduate Internship 2026", "Capitec Bank Branch Learnership 2026", "Eskom Engineering Learnership 2026", "KwaZulu-Natal Department of Health Internship 2026", "MerSETA Boilermaker Apprenticeship 2026", "MICT SETA ICT Learnership 2026", "NSFAS Bursary 2026 — University & TVET Funding", "Old Mutual Graduate Internship 2026", "Sasol Engineering & Science Bursary 2026", "Shoprite Group Retail Learnership 2026", "Standard Bank YES Programme Internship 2026", "Transnet Freight Rail Apprenticeship 2026"]
- `updated_date` unique values (10): ["2026-04-05", "2026-04-08", "2026-04-10", "2026-04-12", "2026-04-14", "2026-04-15", "2026-04-16", "2026-04-18", "2026-04-20", "2026-04-22"]

### Boolean normalization audit

- No boolean representation inconsistencies detected.

### Date format analysis

- `closing_date`: {'YYYY-MM-DD': 12} — OK: all values use YYYY-MM-DD.
- `posted_date`: {'YYYY-MM-DD': 12} — OK: all values use YYYY-MM-DD.
- `updated_date`: {'YYYY-MM-DD': 12} — OK: all values use YYYY-MM-DD.
- Required standard: YYYY-MM-DD.

### URL validation audit

- `source_url`: 0 missing, 0 invalid, 0 non-HTTPS, 0 duplicate URL values.
- `application_url`: 0 missing, 0 invalid, 0 non-HTTPS, 0 duplicate URL values.

### Duplicate structure detection

- No duplicates found by title + closing_date or source_url methods.

## Schema inventory: `Data/guides.json`

### Fields found

- body
- category
- faq
- id
- intro
- meta_description
- meta_title
- published_date
- read_time
- related_slugs
- slug
- title
- updated_date

### Field types

- body → string
- category → string
- faq → array
- id → string
- intro → string
- meta_description → string
- meta_title → string
- published_date → string
- read_time → string
- related_slugs → array
- slug → string
- title → string
- updated_date → string

### Required fields (present in 95%+ records)

- body
- category
- faq
- id
- intro
- meta_description
- meta_title
- published_date
- read_time
- related_slugs
- slug
- title
- updated_date

### Optional fields (present in <95% records)

- None

### Inconsistent presence

- None detected.

### Field usage frequency

- body → 100.0%
- category → 100.0%
- faq → 100.0%
- id → 100.0%
- intro → 100.0%
- meta_description → 100.0%
- meta_title → 100.0%
- published_date → 100.0%
- read_time → 100.0%
- related_slugs → 100.0%
- slug → 100.0%
- title → 100.0%
- updated_date → 100.0%

### String value variation analysis

- `body` unique values (7): ["<h2>CV Structure for Learnership Applicants</h2><p><strong>Section 1 — Personal Details:</strong> Full name, ID number, contact number, email address, physical address, province. Do not include a photo unless specifically requested.</p><p><strong>Section 2 — Education:</strong> Institution name, year completed, relevant subjects and symbols. List Matric first. If you have a TVET qualification or any other certificate, list it too.</p><p><strong>Section 3 — Experience:</strong> Any work experience, even informal, part-time, or volunteer. Include company name, your role, dates, and 2–3 bullet points. No experience? Include community work or school projects.</p><p><strong>Section 4 — Skills:</strong> Basic computer literacy, languages spoken, driver's licence if applicable. Keep it brief and honest.</p><p><strong>Section 5 — References:</strong> Two references — a teacher, community leader, or family friend (not a parent). Include name, relationship, and phone number.</p><h2>Non-Negotiable CV Rules</h2><p>Maximum 2 pages. Font: Arial or Calibri, size 11–12. No spelling errors. Save as PDF — not Word (.doc). File name: YourName-CV.pdf. No photo unless requested.</p>", "<h2>Step 1: Check Your Eligibility</h2><p>Confirm you meet the requirements: South African ID, Matric certificate, unemployed status, and the correct age range. Some learnerships require specific Matric subjects.</p><h2>Step 2: Gather Your Documents</h2><p>You will need: certified copy of your ID (within 3 months), certified Matric certificate, CV of maximum 2 pages, motivational letter of 1 page, and proof of residence not older than 3 months.</p><h2>Step 3: Write a Strong Motivational Letter</h2><p>Most applicants fail at this step by sending a generic letter. Your letter must name the specific company and programme, explain why you want this particular opportunity, and describe what you bring to it. Tailor every letter — generic ones are rejected.</p><h2>Step 4: Submit via the Official Website</h2><p>Apply only through the company's official careers page. Email subject format: <strong>Application: [Learnership Name] — [Your Full Name]</strong>. Send before the closing date — late applications are never considered.</p><h2>Step 5: Follow Up Once</h2><p>One week after applying, send a short follow-up email confirming your application. Once only — multiple follow-ups create a negative impression.</p>", "<h2>Who Qualifies for NSFAS in 2026?</h2><p>You qualify if: you are a South African citizen, your household income is R350 000 or less per year, and you are registered or planning to register at a public university or TVET college. SASSA grant recipients qualify automatically — no income proof needed.</p><h2>What Does NSFAS Cover?</h2><p>For university students: tuition fees, accommodation (in university residences), meal allowance, transport allowance, and personal care allowance. For TVET college students: tuition and a monthly living allowance.</p><h2>How to Apply</h2><p>Applications open October–November each year at <strong>myNSFAS.org.za</strong> — the only legitimate application portal. Never use agents claiming to apply on your behalf. Any agent charging a fee is running a scam. NSFAS applications are free.</p><h2>NSFAS vs Private Bursary</h2><p>Apply for both — but double funding is not allowed. If you receive a full corporate bursary covering all costs, you cannot also receive NSFAS. Disclose all funding applications honestly on each form.</p>", "<p>Learnerships are funded through South Africa's SETA system. When a company runs a learnership, they receive a government grant to cover the cost of training and employing a learner. This is why learnerships exist: government incentivises companies to train people who would otherwise not get work experience.</p><h2>How Long Does a Learnership Last?</h2><p>Most learnerships run for 12 months. Some technical learnerships in engineering, manufacturing, and mining can run 18 to 24 months. At the end, you receive a SAQA-registered NQF qualification.</p><h2>What Is a Learnership Stipend?</h2><p>A stipend is the monthly allowance paid to learners during training. In 2026, most learnerships pay between R3 500 and R8 000 per month, depending on the sector and NQF level. Stipends are not subject to PAYE tax.</p><h2>Who Can Apply?</h2><p>Most learnerships require: South African citizen with a valid ID, Matric certificate, currently unemployed, aged 18–35. Some require specific subjects. Check the eligibility section of each listing carefully.</p>", "<p>SETAs are funded by a 1% payroll levy that every employer with a large enough payroll pays to SARS. This money goes into a Skills Development Fund. SETAs use this fund to pay companies to train unemployed people through learnerships, apprenticeships, and skills programmes.</p><h2>How Do SETAs Help Job Seekers?</h2><p>When a company runs a SETA-funded learnership, they take on unemployed people as learners. You receive on-the-job training AND a nationally recognised qualification — free of charge, with a monthly stipend. You never apply to the SETA directly; you apply to companies running SETA-funded programmes.</p><h2>Which SETA Covers Which Sector?</h2><p>Each SETA covers a specific industry. For IT learnerships, look for MICT SETA programmes. For banking, BANKSETA is relevant. For construction, CETA. For retail, W&RSETA. You'll find the SETA name listed on every opportunity in our directory.</p><h2>Do I Need to Pay Anything?</h2><p><strong>No.</strong> All legitimate SETA-funded learnerships are completely free of charge. If anyone asks you to pay an application fee, registration fee, or any deposit — it is a scam. Report it to the SAPS immediately.</p>", "<p>The most important rule: <strong>legitimate learnerships in South Africa are ALWAYS free to apply for.</strong> Any programme that asks you to pay anything before you start is a scam.</p><h2>7 Warning Signs of a Learnership Scam</h2><p><strong>1. An application fee is requested.</strong> No legitimate SETA-funded learnership ever charges a fee to apply.</p><p><strong>2. Documents requested before an interview.</strong> Real programmes only request documents at the interview stage or after an offer.</p><p><strong>3. The stipend is unusually high.</strong> A learnership advertising R25 000/month is almost certainly fake. Typical stipends are R3 500 to R8 000.</p><p><strong>4. WhatsApp-only contact from an unknown number.</strong> Legitimate employers contact applicants via official company email addresses.</p><p><strong>5. No official company website.</strong> Always verify the company exists at cipc.co.za before engaging.</p><p><strong>6. Urgency pressure.</strong> Apply in the next 24 hours or lose your spot. Legitimate programmes have published closing dates, not artificial urgency.</p><p><strong>7. The email domain is generic.</strong> setalearnerships@gmail.com is not an official company email. Real employers use @companyname.co.za addresses.</p><h2>How to Verify a Legitimate Opportunity</h2><p>Search the company name on Google and go directly to their official website. Look for the opportunity on their official careers page. If it's not there, treat it as suspicious. Verify company registration at cipc.co.za.</p>", "<p>The National Qualifications Framework (NQF) is South Africa's system for classifying all education and training qualifications. Every qualification sits at a specific level. This matters because every opportunity listing states a minimum NQF level or qualification type.</p><h2>The Key Levels</h2><p><strong>NQF Level 4 — Matric (Grade 12 / NSC).</strong> The most common entry level for learnerships. Most SETA learnership programmes lead to an NQF Level 4 qualification.</p><p><strong>NQF Level 5 — Higher Certificate.</strong> A one-year post-Matric qualification. Opens more learnership and some internship doors.</p><p><strong>NQF Level 6 — Diploma / Advanced Certificate.</strong> A three-year TVET or university diploma. Required for many graduate internship programmes.</p><p><strong>NQF Level 7 — Bachelor's Degree.</strong> A three-year university degree. Required for corporate graduate programmes and professional internships.</p><h2>TVET and Learnerships</h2><p>Many SETA learnerships specifically target TVET graduates — particularly N6 graduates who need the 18-month practical component for their National Diploma. A TVET-funded learnership allows you to complete your diploma while being paid a stipend.</p>"]
- `category` unique values (2): ["career-advice", "seta-guides"]
  - Normalized suggestion: ["career-advice", "seta-guides"]
- `id` unique values (7): ["g001", "g002", "g003", "g004", "g005", "g006", "g007"]
- `intro` unique values (7): ["A learnership is a structured training programme combining classroom learning with real work experience. You earn a nationally recognised qualification AND a monthly stipend — and it's completely free to apply for.", "A SETA — Sector Education and Training Authority — is a government body that funds skills training in South Africa. There are 21 SETAs, each responsible for a different sector of the economy.", "Applying for a learnership doesn't have to be complicated. This step-by-step guide covers everything — from checking your eligibility to submitting your application. Never pay to apply.", "Learnership scams are widespread in South Africa. Fake opportunities advertised on social media cost job seekers thousands of rands every year. Here's how to protect yourself.", "NSFAS covers tuition, accommodation, and a living allowance for students at public universities and TVET colleges. No repayment required for most students.", "The NQF — National Qualifications Framework — classifies all South African qualifications from Level 1 to Level 10. Understanding it helps you know exactly which opportunities you qualify for.", "Your CV is the first filter between you and a learnership. A clean, 2-page CV with the right information gives you a real advantage over hundreds of applications."]
- `meta_description` unique values (7): ["A learnership pays you a monthly stipend while you earn an NQF qualification. Find out how learnerships work, who can apply, and how to get one in South Africa.", "Complete step-by-step guide to applying for learnerships in South Africa. Documents needed, motivational letter tips, and how to find official applications.", "Everything you need to know about NSFAS 2026 — who qualifies, what is covered, how to apply at myNSFAS, and how it compares to private bursaries.", "Learn to identify fake learnerships in South Africa. Know the 7 red flags, how to verify opportunities, and what to do if you've been scammed.", "Learn what a SETA is, how SETAs fund learnerships, and how to access SETA opportunities in South Africa. Clear, plain-language guide updated for 2026.", "Understand South Africa's NQF qualification levels and what they mean for learnerships, bursaries, and career opportunities. Plain-language guide.", "Write a winning CV for learnership applications in South Africa. Structure, rules, and tips — free guide for first-time applicants."]
- `meta_title` unique values (7): ["How to Apply for a Learnership in South Africa: Step-by-Step 2026", "How to Avoid Learnership Scams in South Africa 2026", "How to Write a CV for a Learnership — South Africa 2026", "NQF Levels Explained — South Africa 2026 Guide", "NSFAS 2026 Application: Who Qualifies, What It Covers & How to Apply", "What Is a Learnership? South Africa Guide 2026", "What Is a SETA in South Africa? Complete 2026 Guide"]
- `published_date` unique values (7): ["2026-04-01", "2026-04-02", "2026-04-03", "2026-04-04", "2026-04-05", "2026-04-06", "2026-04-07"]
- `read_time` unique values (3): ["4 min", "5 min", "6 min"]
- `slug` unique values (7): ["cv-for-learnership", "how-to-apply-for-learnerships", "learnership-scams", "nqf-levels-explained", "nsfas-2026", "what-is-a-learnership", "what-is-a-seta"]
- `title` unique values (7): ["How to Apply for Learnerships: Step-by-Step Guide 2026", "How to Avoid Learnership Scams in South Africa (2026)", "How to Write a CV for a Learnership (With Structure)", "NQF Levels Explained: What They Mean for Your Career (2026)", "NSFAS 2026: How to Apply, Who Qualifies & What It Covers", "What Is a Learnership in South Africa? 2026 Explained", "What Is a SETA in South Africa? Complete Guide 2026"]
- `updated_date` unique values (7): ["2026-04-02", "2026-04-03", "2026-04-04", "2026-04-05", "2026-04-06", "2026-04-07", "2026-04-20"]

### Boolean normalization audit

- No boolean representation inconsistencies detected.

### Date format analysis

- `updated_date`: {'YYYY-MM-DD': 7} — OK: all values use YYYY-MM-DD.
- Required standard: YYYY-MM-DD.

### URL validation audit

- No target URL fields (`source_url`, `application_url`) detected.

### Duplicate structure detection

- No duplicates found by title + closing_date or source_url methods.

## Schema inventory: `Data/categories.json`

### Fields found

- bg
- color
- description
- icon
- id
- label
- plural
- seo_desc
- seo_title

### Field types

- bg → string
- color → string
- description → string
- icon → string
- id → string
- label → string
- plural → string
- seo_desc → string
- seo_title → string

### Required fields (present in 95%+ records)

- bg
- color
- description
- icon
- id
- label
- plural
- seo_desc
- seo_title

### Optional fields (present in <95% records)

- None

### Inconsistent presence

- None detected.

### Field usage frequency

- bg → 100.0%
- color → 100.0%
- description → 100.0%
- icon → 100.0%
- id → 100.0%
- label → 100.0%
- plural → 100.0%
- seo_desc → 100.0%
- seo_title → 100.0%

### String value variation analysis

- `bg` unique values (4): ["#dbeafe", "#dcfce7", "#f3e8ff", "#fef3c7"]
- `color` unique values (4): ["#166534", "#1e40af", "#6b21a8", "#92400e"]
- `description` unique values (4): ["Artisan trade training leading to a nationally recognised trade certificate. 2–3 year programmes.", "Earn a nationally recognised qualification while earning a monthly stipend. Free to apply.", "Financial funding for studies at a public university or TVET college. Covers tuition and living costs.", "Structured work experience for graduates and students. Government and private sector programmes."]
- `icon` unique values (4): ["🎓", "💼", "📚", "🔧"]
- `id` unique values (4): ["apprenticeship", "bursary", "internship", "learnership"]
- `label` unique values (4): ["Apprenticeships", "Bursaries", "Internships", "Learnerships"]
- `plural` unique values (4): ["apprenticeships", "bursaries", "internships", "learnerships"]
- `seo_desc` unique values (4): ["Find apprenticeship programmes in South Africa for 2026. Electrical, mechanical, boilermaking, plumbing and more. Verified SETA-funded opportunities.", "Find government and corporate bursaries for South African students in 2026. NSFAS, merit bursaries, and need-based funding. Updated daily.", "Find verified learnerships across South Africa for 2026. Filter by province, qualification, and closing date. All free to apply. Updated daily.", "Graduate internships and work experience programmes across South Africa for 2026. Filter by province, sector, and qualification. Verified listings."]
- `seo_title` unique values (4): ["Apprenticeships in South Africa 2026 — Become a Qualified Artisan", "Bursaries South Africa 2026 — Find Funding for Your Studies", "Internships in South Africa 2026 — Graduate Opportunities", "Learnerships in South Africa 2026 — Find & Apply"]

### Boolean normalization audit

- No boolean representation inconsistencies detected.

### Date format analysis

- No target date fields (`closing_date`, `posted_date`, `updated_date`) detected.

### URL validation audit

- No target URL fields (`source_url`, `application_url`) detected.

### Duplicate structure detection

- No duplicates found by title + closing_date or source_url methods.

## Schema inventory: `Data/provinces.json`

### Fields found

- description
- icon
- id
- label
- major_cities
- seo_desc
- seo_title
- top_employers

### Field types

- description → string
- icon → string
- id → string
- label → string
- major_cities → array
- seo_desc → string
- seo_title → string
- top_employers → array

### Required fields (present in 95%+ records)

- description
- icon
- id
- label
- major_cities
- seo_desc
- seo_title
- top_employers

### Optional fields (present in <95% records)

- None

### Inconsistent presence

- None detected.

### Field usage frequency

- description → 100.0%
- icon → 100.0%
- id → 100.0%
- label → 100.0%
- major_cities → 100.0%
- seo_desc → 100.0%
- seo_title → 100.0%
- top_employers → 100.0%

### String value variation analysis

- `description` unique values (9): ["Agriculture, gold mining near Welkom, and public sector employment in Bloemfontein (judicial capital) drive opportunities. Mangaung Metro is the largest public sector employer.", "Centre of South Africa's platinum mining industry. Sibanye-Stillwater, Lonmin, and Anglo American run major learnership and apprenticeship programmes in the Rustenburg area.", "Home to Africa's busiest port, driving consistent Transnet and logistics opportunities. Strong automotive (Toyota SA), sugar, and petrochemicals sectors.", "Mining (platinum, chrome, coal) and agriculture drive the economy. Anglo American Platinum, Implats, and Northam Platinum run consistent learnership and apprenticeship programmes.", "South Africa's automotive manufacturing hub. Volkswagen SA (Uitenhage), Isuzu (East London), and Mercedes-Benz SA are major learnership and apprenticeship providers.", "South Africa's economic heartland. The highest volume of learnerships, internships, and bursaries of any province. Home to the big banks, Eskom, Transnet, and most major corporate headquarters.", "South Africa's energy heartland — home to most of Eskom's power stations and Sasol's Secunda operations. Consistent source of engineering and instrumentation learnership opportunities.", "South Africa's largest but least populated province. Diamond mining (De Beers) and iron ore (Kumba Iron Ore) are primary opportunity sources. Less competition per listing than any other province.", "South Africa's second-largest opportunity market. Cape Town hosts major retail headquarters (Shoprite, Pick n Pay, Woolworths), Capitec Bank HQ, and a growing technology sector."]
- `icon` unique values (8): ["⚡", "🌊", "🌻", "🌾", "🌿", "🏔️", "🏙️", "🏜️"]
- `id` unique values (9): ["eastern-cape", "free-state", "gauteng", "kwazulu-natal", "limpopo", "mpumalanga", "north-west", "northern-cape", "western-cape"]
- `label` unique values (9): ["Eastern Cape", "Free State", "Gauteng", "KwaZulu-Natal", "Limpopo", "Mpumalanga", "North West", "Northern Cape", "Western Cape"]
- `seo_desc` unique values (9): ["Find learnerships, bursaries, and internships in KwaZulu-Natal for 2026. Durban, Pietermaritzburg and more. Verified opportunities.", "Find learnerships, bursaries, and internships in the Eastern Cape for 2026. Automotive, government, and manufacturing opportunities. Verified.", "Find verified learnerships and internships in Mpumalanga for 2026. Energy, mining, and government opportunities.", "Find verified learnerships and internships in North West province for 2026. Mining, government, and engineering opportunities.", "Find verified learnerships and internships in the Free State for 2026. Government, agriculture, and mining opportunities. Verified listings.", "Find verified learnerships and internships in the Northern Cape for 2026. Mining, government, and artisan opportunities. Less competition.", "Find verified learnerships, bursaries, and internships in Gauteng for 2026. Johannesburg, Pretoria, and Ekurhuleni. Filter by category and closing date.", "Find verified learnerships, bursaries, and internships in Limpopo for 2026. Mining, agriculture, and government opportunities.", "Find verified learnerships, bursaries, and internships in the Western Cape for 2026. Cape Town and surrounding areas. Verified, free to apply."]
- `seo_title` unique values (9): ["Learnerships & Bursaries in Free State 2026 | Bloemfontein", "Learnerships & Bursaries in Limpopo 2026 | Polokwane", "Learnerships & Bursaries in Mpumalanga 2026 | Nelspruit & Secunda", "Learnerships & Bursaries in North West 2026 | Rustenburg", "Learnerships & Bursaries in Northern Cape 2026 | Kimberley", "Learnerships & Internships in Eastern Cape 2026 | Gqeberha & East London", "Learnerships & Internships in Gauteng 2026 | Johannesburg & Pretoria", "Learnerships & Internships in KwaZulu-Natal 2026 | Durban", "Learnerships & Internships in Western Cape 2026 | Cape Town"]

### Boolean normalization audit

- No boolean representation inconsistencies detected.

### Date format analysis

- No target date fields (`closing_date`, `posted_date`, `updated_date`) detected.

### URL validation audit

- No target URL fields (`source_url`, `application_url`) detected.

### Duplicate structure detection

- No duplicates found by title + closing_date or source_url methods.

## Schema inconsistencies and data quality issues

- Exact lowercase `/data/...` paths are missing, while audited files exist under `Data/...`; this may break code or automation expecting lowercase paths.

## Normalization requirements

- Enforce lowercase canonical category IDs where `category` is used.
- Enforce canonical province labels or IDs before adding filter automation.
- Enforce YYYY-MM-DD for all date fields.
- Enforce HTTPS URLs for `source_url` and `application_url` when present.
- Keep boolean fields as real JSON booleans, not strings or numbers.

## Recommended fixes (no implementation performed)

1. Decide whether the canonical data directory should be `data/` or `Data/`, then update repository paths consistently in a future change.
2. Adopt `docs/schema-draft.json` as the preliminary schema contract for automation planning.
3. Add validation checks for required fields, allowed values, dates, URLs, duplicate URLs, and duplicate title/date combinations.
4. Continue monitoring duplicate `source_url` values in future audits; none were detected in this baseline.
5. Keep this audit as a baseline before any normalization or cleanup phase.
````

## File: docs/schema-draft.json
````json
{
  "opportunities": {
    "required_fields": [
      "application_url",
      "category",
      "closing_date",
      "description",
      "documents_needed",
      "eligibility",
      "expired",
      "featured",
      "id",
      "posted_date",
      "province",
      "qualification_level",
      "sector",
      "slug",
      "source_name",
      "source_url",
      "stipend",
      "tags",
      "title",
      "updated_date",
      "verified"
    ],
    "optional_fields": [],
    "field_types": {
      "application_url": "string",
      "category": "string",
      "closing_date": "string",
      "description": "string",
      "documents_needed": "array",
      "eligibility": "array",
      "expired": "boolean",
      "featured": "boolean",
      "id": "string",
      "posted_date": "string",
      "province": "string",
      "qualification_level": "string",
      "sector": "string",
      "slug": "string",
      "source_name": "string",
      "source_url": "string",
      "stipend": "string",
      "tags": "array",
      "title": "string",
      "updated_date": "string",
      "verified": "boolean"
    },
    "allowed_values": {
      "province": [
        "Gauteng",
        "KwaZulu-Natal",
        "Nationwide",
        "Western Cape"
      ],
      "category": [
        "apprenticeship",
        "bursary",
        "internship",
        "learnership"
      ]
    },
    "format_rules": {
      "dates": "YYYY-MM-DD",
      "urls": "https only"
    }
  },
  "guides": {
    "required_fields": [
      "body",
      "category",
      "faq",
      "id",
      "intro",
      "meta_description",
      "meta_title",
      "published_date",
      "read_time",
      "related_slugs",
      "slug",
      "title",
      "updated_date"
    ],
    "optional_fields": [],
    "field_types": {
      "body": "string",
      "category": "string",
      "faq": "array",
      "id": "string",
      "intro": "string",
      "meta_description": "string",
      "meta_title": "string",
      "published_date": "string",
      "read_time": "string",
      "related_slugs": "array",
      "slug": "string",
      "title": "string",
      "updated_date": "string"
    },
    "allowed_values": {
      "category": [
        "career-advice",
        "seta-guides"
      ]
    },
    "format_rules": {
      "dates": "YYYY-MM-DD",
      "urls": "https only"
    }
  },
  "categories": {
    "required_fields": [
      "bg",
      "color",
      "description",
      "icon",
      "id",
      "label",
      "plural",
      "seo_desc",
      "seo_title"
    ],
    "optional_fields": [],
    "field_types": {
      "bg": "string",
      "color": "string",
      "description": "string",
      "icon": "string",
      "id": "string",
      "label": "string",
      "plural": "string",
      "seo_desc": "string",
      "seo_title": "string"
    },
    "allowed_values": {},
    "format_rules": {
      "dates": "YYYY-MM-DD",
      "urls": "https only"
    }
  },
  "provinces": {
    "required_fields": [
      "description",
      "icon",
      "id",
      "label",
      "major_cities",
      "seo_desc",
      "seo_title",
      "top_employers"
    ],
    "optional_fields": [],
    "field_types": {
      "description": "string",
      "icon": "string",
      "id": "string",
      "label": "string",
      "major_cities": "array",
      "seo_desc": "string",
      "seo_title": "string",
      "top_employers": "array"
    },
    "allowed_values": {},
    "format_rules": {
      "dates": "YYYY-MM-DD",
      "urls": "https only"
    }
  }
}
````

## File: docs/source-governance.md
````markdown
# Source Governance

This document defines how the platform manages source lifecycle, crawl cadence, and registry priority. The source registry is stored at `data/sources.json` and is validated by `npm run source:audit`.

## Source lifecycle

The current registry stores lifecycle state with the `active` boolean. Operationally, sources should be handled with the following lifecycle states:

| Lifecycle state | Registry representation | Meaning | Required action |
| --- | --- | --- | --- |
| `active` | `active: true` | The source is approved for discovery, health checks, and future crawler work. | Keep the URL and notes current. Review according to its frequency and priority. |
| `inactive` | `active: false` | The source is temporarily excluded from crawling, usually due to stale content, duplicated coverage, or maintenance. | Keep the entry for auditability. Add a note explaining why it is inactive and when it should be reviewed. |
| `deprecated` | `active: false` | The source has been replaced by another source or no longer publishes useful opportunities. | Keep the entry only when historical traceability matters. Add the replacement `source_id` in `notes` when available. |
| `blocked` | `active: false` | The source should not be crawled because of robots, legal, abuse-prevention, authentication, or repeated blocking concerns. | Do not crawl. Document the reason in `notes` and revisit only after a governance review. |

## Frequency values

Use these exact values in `frequency`:

| Value | Meaning | Typical use |
| --- | --- | --- |
| `hourly` | Check many times per day. | Critical API/RSS feeds that change frequently and are safe to poll. |
| `daily` | Check once per day. | High-signal pages with frequent new opportunities. |
| `weekly` | Check once per week. | Most careers pages, SETA notices, bursary pages, and municipal vacancy listings. |
| `monthly` | Check once per month. | Low-change reference pages or sources that only publish periodic rounds. |

## Priority values

Use these exact numeric values in `priority`:

| Priority | Label | Meaning |
| --- | --- | --- |
| `1` | Critical | Highest-value sources for South African careers, public vacancies, bursaries, learnerships, or internships. |
| `2` | High | Strong source with recurring opportunities or broad national/provincial relevance. |
| `3` | Medium | Useful source with regional, sector-specific, or less frequent opportunities. |
| `4` | Low | Opportunistic or low-frequency source kept for coverage completeness. |

## Governance rules

1. Every source must have a stable, kebab-case `source_id` that is never reused for a different organisation.
2. `type` must be one of `government`, `seta`, `university`, `municipality`, `soe`, or `private`.
3. `crawl_strategy` must be one of `html`, `pdf`, `rss`, `api`, or `hybrid`.
4. `frequency` must be one of `hourly`, `daily`, `weekly`, or `monthly`.
5. `priority` must be an integer from `1` to `4`.
6. `last_checked` and `last_success` must be `null` until automation records ISO-8601 UTC timestamps.
7. Material source changes should be documented in `notes`, especially lifecycle changes and blocked/deprecated reasons.
````

## File: docs/source-registry.md
````markdown
# Source Registry

The source registry is the canonical list of organisations and pages that the platform can audit, monitor, and eventually crawl for South African career opportunities. It is intentionally separate from published opportunity content so new ingestion tooling can be built without changing the current website data model.

## Registry location

```text
data/sources.json
```

The file is a JSON array of source objects. Every object must follow the same schema so scripts can validate, health-check, and later crawl sources consistently.

## Source schema

```json
{
  "source_id": "dpsa",
  "name": "Department of Public Service and Administration (DPSA)",
  "type": "government",
  "url": "https://www.dpsa.gov.za/newsroom/psvc/",
  "active": true,
  "crawl_strategy": "pdf",
  "frequency": "weekly",
  "priority": 1,
  "last_checked": null,
  "last_success": null,
  "notes": "Public Service Vacancy Circulars."
}
```

## Field meanings

| Field | Type | Required | Meaning | Validation |
| --- | --- | --- | --- | --- |
| `source_id` | string | Yes | Stable machine identifier used in reports, logs, future crawler records, and deduplication. | Must be unique and kebab-case: lowercase letters, numbers, and single hyphens only. |
| `name` | string | Yes | Human-readable organisation or source name. | Must be a non-empty string. |
| `url` | string | Yes | Best landing page for vacancies, bursaries, learnerships, internships, supplier notices, or programme announcements. | Must be a unique absolute `http` or `https` URL. |
| `type` | string | Yes | Source category used for coverage reporting and governance. | Must be one of the approved source types below. |
| `active` | boolean | Yes | Whether this source is eligible for health checks and future crawler runs. | Must be `true` or `false`. |
| `crawl_strategy` | string | Yes | Expected extraction approach for future ingestion. | Must be one of the approved crawl strategies below. |
| `frequency` | string | Yes | Intended review or crawl cadence. | Must be one of the approved frequencies below. |
| `priority` | number | Yes | Relative source importance. `1` is highest priority and `4` is lowest. | Must be `1`, `2`, `3`, or `4`. |
| `last_checked` | string/null | Yes | Timestamp reserved for automation to record the most recent probe or crawl attempt. | Must be `null` or an ISO-8601 UTC timestamp. |
| `last_success` | string/null | Yes | Timestamp reserved for automation to record the most recent successful probe or crawl. | Must be `null` or an ISO-8601 UTC timestamp. |
| `notes` | string | Yes | Operational context: what the source publishes, special handling, lifecycle notes, or replacement source IDs. | Must be a string; empty notes produce a warning. |

## Approved enum values

### Source types

- `government` — national or provincial departments, public agencies, and government programmes.
- `seta` — Sector Education and Training Authorities.
- `university` — universities and universities of technology.
- `municipality` — metropolitan, district, or local municipalities.
- `soe` — state-owned enterprises and public companies.
- `private` — private employers, banks, retailers, telcos, industrial firms, and similar organisations.

### Crawl strategies

- `html` — content is primarily on standard web pages.
- `pdf` — content is primarily published in PDFs.
- `rss` — content is available through RSS/Atom feeds.
- `api` — content is available through a structured API.
- `hybrid` — source requires multiple approaches, such as HTML pages plus PDF attachments or external career portals.

### Frequencies

- `hourly` — high-volume feeds that are safe and valuable to poll multiple times per day.
- `daily` — high-signal sources with frequent postings.
- `weekly` — normal cadence for most careers pages, SETA notices, and government vacancy pages.
- `monthly` — low-change sources or periodic opportunity rounds.

## Onboarding process

1. **Identify the source.** Confirm the organisation publishes opportunities relevant to the platform: jobs, bursaries, learnerships, apprenticeships, internships, graduate programmes, tenders, or career guidance.
2. **Choose the canonical URL.** Prefer the most specific public landing page that lists opportunities. Avoid search results, login-only pages, expired campaign pages, and third-party mirrors unless they are the official application portal.
3. **Assign a stable `source_id`.** Use a concise kebab-case ID. Do not reuse IDs from removed or deprecated sources.
4. **Classify the source.** Pick the closest `type`, `crawl_strategy`, `frequency`, and `priority` values from the approved enums.
5. **Add operational notes.** Explain what the source publishes and record any known handling requirements, such as PDFs, external applicant tracking systems, or blocked automation.
6. **Validate locally.** Run `npm run source:audit` and resolve every `FAIL` before merging. Review any `WARN` output and update notes or metadata where appropriate.
7. **Probe health.** Run `npm run source:health` to generate a fresh health report for active sources. Non-2xx responses are warnings by default because some public websites block automated probes.
8. **Review reports.** Confirm `reports/source-audit-report.json` and `reports/source-health-report.json` were regenerated and contain expected results.

## Deactivation process

Use deactivation when a source should remain in the registry for auditability but should not be crawled or health-checked.

1. Set `active` to `false`.
2. Keep `source_id`, `name`, `url`, and `type` unchanged unless the organisation itself changed.
3. Update `notes` with the deactivation reason, date, and any replacement `source_id`.
4. Leave `last_checked` and `last_success` unchanged unless an automation process updates them.
5. Run `npm run source:audit` to verify the inactive source still follows schema rules.
6. Do not delete a source unless there is no governance, audit, or historical reason to retain it.

Common deactivation reasons include duplicated coverage, discontinued opportunity pages, permanent redirects to a better source, legal or robots concerns, login-only access, repeated blocking, or source content no longer matching the platform scope.

## Validation rules

The reusable validator lives at:

```text
scripts/validators/source-validator.js
```

It enforces these rules for `source-audit.js`, `source-health-check.js`, and future crawler code:

- The registry root must be an array.
- Required fields must be present: `source_id`, `name`, `url`, `type`, `active`, `crawl_strategy`, `frequency`, `priority`, `last_checked`, `last_success`, and `notes`.
- `source_id` values must be unique and kebab-case.
- `url` values must be valid absolute `http` or `https` URLs and must be unique after normalization.
- `type`, `frequency`, `crawl_strategy`, and `priority` must match approved enum values.
- `name` must be non-empty.
- `active` must be boolean.
- `last_checked` and `last_success` must be `null` or ISO-8601 UTC timestamps.
- `notes` must be a string; blank notes are reported as warnings.

## Operational scripts and reports

Run the registry audit:

```bash
npm run source:audit
```

The audit prints a `PASS`, `WARN`, or `FAIL` summary and writes:

```text
reports/source-audit-report.json
```

Run the active-source health check:

```bash
npm run source:health
```

The health check records HTTP status, redirects, response time, SSL availability, and overall health for every active source, then writes:

```text
reports/source-health-report.json
```

The health command exits successfully for warnings unless strict mode is enabled:

```bash
SOURCE_HEALTH_STRICT=1 npm run source:health
```

## Crawler rejection reasons

The crawler preserves the existing rejected-record shape: each rejection still includes `source_id`, `url`, `reason`, optional `details`, and the timestamp added by the staging pipeline. Source registry validation failures now use a specific reason so operators can distinguish bad registry metadata from fetched content validation failures.

| Reason | Stage | Meaning |
| --- | --- | --- |
| `INVALID_SOURCE_REGISTRY` | Source loader | An active source failed `scripts/validators/source-validator.js` checks before crawl processing. |
| `FETCH_FAILED` | Fetcher | The source could not be fetched after configured retries. |
| `UNSUPPORTED_CONTENT_TYPE` | Router | The fetched response could not be routed to a supported parser. |
| `PARSER_ERROR` | Parser | Supported content was fetched, but parsing threw an error. |
| `VALIDATION_FAILED` | Rejection pipeline | Parsed and normalized content failed downstream content validation. |

Example source registry rejection:

```json
{
  "timestamp": "2026-06-16T00:00:00.000Z",
  "source_id": "sources[4]",
  "url": "",
  "reason": "INVALID_SOURCE_REGISTRY",
  "details": [
    "sources[4].source_id must be a non-empty kebab-case identifier.",
    "sources[4].url must be a non-empty string."
  ]
}
```

Example crawl report rejection summary:

```json
{
  "rejected_records": 2,
  "rejected_by_reason": {
    "INVALID_SOURCE_REGISTRY": 1,
    "FETCH_FAILED": 1
  }
}
```
````

## File: docs/UPSCALE_WORKFLOW.md
````markdown
# OpportunitiesZA Upscale Workflow

This workflow turns the current static site into a repeatable content and quality operation that can scale from a manually maintained directory into a trusted, searchable opportunity platform.

## North-star outcome

Grow OpportunitiesZA by increasing **verified active listings**, **useful career guides**, and **organic search coverage** without sacrificing trust. Every workflow below protects one of three outcomes:

1. **Trust:** no paid-to-apply scams, stale deadlines, duplicate listings, or broken source links.
2. **Reach:** each verified listing and guide creates clean SEO surfaces through the sitemap and route structure.
3. **Speed:** contributors can add opportunities quickly while automated checks catch avoidable mistakes.

## Operating cadence

| Cadence | Owner | Workflow | Definition of done |
| --- | --- | --- | --- |
| Daily | Content curator | Find and verify new opportunities from official employer, SETA, university, or government sources. | At least 3 new or refreshed records in `data/opportunities.json`, with source URLs and updated dates. |
| Daily | Content curator | Expiry sweep. | Listings with closing dates in the past are marked `expired: true` or updated from official sources. |
| Twice weekly | Editor | Publish or refresh guides. | One guide in `data/guides.json` is added or materially improved with a new `updated_date`. |
| Weekly | Maintainer | Quality release. | `npm run check` passes, sitemap changes are committed, and GitHub Pages deploys from a clean main branch. |
| Monthly | Product owner | Growth review. | Review search queries, top categories, missing provinces, and conversion clicks; reprioritise the backlog. |

## Contributor workflow

1. **Create a focused branch** named `content/<topic>` or `platform/<feature>`.
2. **Edit the smallest responsible file:** opportunities in `data/opportunities.json`, guides in `data/guides.json`, metadata in `data/categories.json` or `data/provinces.json`.
3. **Validate locally:** run `npm run validate` before committing.
4. **Regenerate the sitemap:** run `npm run sitemap` after content changes.
5. **Run link checks:** run `npm run links` when source or application URLs change; review warnings from portals that block bots.
6. **Run the full check:** run `npm run check` and confirm `git diff` only includes intentional changes.
7. **Open a pull request** using the checklist below.
8. **Merge only after quality gates pass** in GitHub Actions.

## Pull request checklist

- [ ] All new opportunities have an official `source_url` and `application_url`.
- [ ] `closing_date`, `posted_date`, and `updated_date` use `YYYY-MM-DD`.
- [ ] Scam-sensitive copy states that legitimate applications are free where relevant.
- [ ] Expired listings are marked `expired: true` instead of deleted, unless removal is intentional.
- [ ] `npm run links` has no hard-broken source or application URLs.
- [ ] `npm run check` passes locally.
- [ ] `sitemap.xml` is regenerated and committed when content changes affect public pages.

## Automation now available

The repository now includes a zero-dependency Node workflow for repeatable quality gates:

```bash
npm run validate
npm run links
npm run sitemap
npm run check
```

GitHub Actions runs the same checks on pushes and pull requests. The workflow validates data quality, checks opportunity URLs, regenerates the sitemap, and fails if generated sitemap changes were not committed.

## Scalable backlog

### Phase 1 — Trust foundation

- Maintain link-checking coverage for `application_url` and `source_url`, including retry/timeout tuning and strict scheduled checks.
- Add duplicate detection for similar opportunity titles across employers and provinces.
- Add an expiry dashboard that groups listings by expired, closing this week, and closing this month.
- Add source verification notes for high-risk listings.

### Phase 2 — Content growth engine

- Add templates for common opportunity types: learnership, bursary, internship, apprenticeship, and TVET pathway.
- Add province landing pages for every province/category combination with enough active inventory.
- Add guide refresh reminders for articles older than 90 days.
- Add a lightweight editorial calendar for seasonal application peaks.

### Phase 3 — Product scale

- Split the single-page application into maintainable modules when feature velocity starts slowing down.
- Add client-side data loading from `data/*.json` so content updates do not require editing `index.html`.
- Add saved filters, deadline reminders, and shareable search URLs.
- Add structured data for opportunities, FAQ pages, breadcrumbs, and organization metadata.

### Phase 4 — Growth and analytics

- Configure privacy-conscious analytics for search terms, apply-button clicks, category pages, and province pages.
- Add Search Console review tasks for indexing gaps, query opportunities, and broken rich results.
- Add a monthly content scorecard: active listings, expired listings, apply clicks, guide traffic, and top missing sectors.
- Build a partner intake process for employers and training providers while preserving manual verification.

## Data quality rules

Automated validation currently enforces:

- Lowercase `data/` directory compatibility for Linux and GitHub Pages.
- Valid JSON arrays for all data files.
- Required fields for opportunities and guides.
- Unique `id` and `slug` values.
- Lowercase kebab-case slugs.
- Valid `YYYY-MM-DD` dates.
- Valid HTTP(S) source and application URLs.
- Boolean values for `verified`, `featured`, and `expired`.
- Non-empty arrays for eligibility, documents, tags, and guide FAQs.

Warnings highlight issues worth editorial review, such as expired closing dates still marked active or meta descriptions longer than search-result best practice.
````

## File: reports/source-audit-report.json
````json
{
  "generated_at": "2026-06-03T20:22:19.494Z",
  "registry_path": "data/sources.json",
  "status": "PASS",
  "summary": {
    "total": 58,
    "active": 58,
    "inactive": 0,
    "by_type": {
      "government": 12,
      "seta": 10,
      "university": 10,
      "soe": 6,
      "private": 10,
      "municipality": 10
    },
    "issue_counts": {
      "PASS": 0,
      "WARN": 0,
      "FAIL": 0
    }
  },
  "issues": []
}
````

## File: reports/source-health-report.json
````json
{
  "generated_at": "2026-06-03T20:22:08.361Z",
  "status": "WARN",
  "summary": {
    "checked": 58,
    "healthy": 0,
    "unhealthy": 58,
    "with_redirects": 0,
    "ssl_available": 0,
    "timeout_ms": 8000
  },
  "sources": [
    {
      "source_id": "dpsa",
      "name": "Department of Public Service and Administration (DPSA)",
      "url": "https://www.dpsa.gov.za/newsroom/psvc/",
      "final_url": "https://www.dpsa.gov.za/newsroom/psvc/",
      "status": null,
      "response_time": 5101,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "fetch failed",
      "checked_at": "2026-06-03T20:21:09.356Z"
    },
    {
      "source_id": "national-treasury",
      "name": "National Treasury",
      "url": "https://www.treasury.gov.za/",
      "final_url": "https://www.treasury.gov.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:12.345Z"
    },
    {
      "source_id": "sars",
      "name": "South African Revenue Service (SARS)",
      "url": "https://www.sars.gov.za/careers/",
      "final_url": "https://www.sars.gov.za/careers/",
      "status": null,
      "response_time": 8003,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:12.349Z"
    },
    {
      "source_id": "department-employment-labour",
      "name": "Department of Employment and Labour",
      "url": "https://www.labour.gov.za/",
      "final_url": "https://www.labour.gov.za/",
      "status": null,
      "response_time": 8003,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:12.352Z"
    },
    {
      "source_id": "department-health",
      "name": "National Department of Health",
      "url": "https://www.health.gov.za/",
      "final_url": "https://www.health.gov.za/",
      "status": null,
      "response_time": 8004,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:12.356Z"
    },
    {
      "source_id": "department-basic-education",
      "name": "Department of Basic Education",
      "url": "https://www.education.gov.za/",
      "final_url": "https://www.education.gov.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:12.358Z"
    },
    {
      "source_id": "mict-seta",
      "name": "MICT SETA",
      "url": "https://www.mict.org.za/",
      "final_url": "https://www.mict.org.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:12.361Z"
    },
    {
      "source_id": "services-seta",
      "name": "Services SETA",
      "url": "https://www.serviceseta.org.za/",
      "final_url": "https://www.serviceseta.org.za/",
      "status": null,
      "response_time": 8000,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:12.363Z"
    },
    {
      "source_id": "etdp-seta",
      "name": "ETDP SETA",
      "url": "https://www.etdpseta.org.za/",
      "final_url": "https://www.etdpseta.org.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:17.358Z"
    },
    {
      "source_id": "bankseta",
      "name": "BANKSETA",
      "url": "https://www.bankseta.org.za/",
      "final_url": "https://www.bankseta.org.za/",
      "status": null,
      "response_time": 8003,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:20.348Z"
    },
    {
      "source_id": "agriseta",
      "name": "AGRISETA",
      "url": "https://www.agriseta.co.za/",
      "final_url": "https://www.agriseta.co.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:20.350Z"
    },
    {
      "source_id": "ceta",
      "name": "Construction Education and Training Authority (CETA)",
      "url": "https://www.ceta.org.za/",
      "final_url": "https://www.ceta.org.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:20.354Z"
    },
    {
      "source_id": "merseta",
      "name": "merSETA",
      "url": "https://www.merseta.org.za/",
      "final_url": "https://www.merseta.org.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:20.357Z"
    },
    {
      "source_id": "lgseta",
      "name": "LGSETA",
      "url": "https://lgseta.org.za/",
      "final_url": "https://lgseta.org.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:20.360Z"
    },
    {
      "source_id": "teta",
      "name": "Transport Education Training Authority (TETA)",
      "url": "https://www.teta.org.za/",
      "final_url": "https://www.teta.org.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:20.362Z"
    },
    {
      "source_id": "hwseta",
      "name": "HWSETA",
      "url": "https://www.hwseta.org.za/",
      "final_url": "https://www.hwseta.org.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:20.365Z"
    },
    {
      "source_id": "unisa",
      "name": "University of South Africa (UNISA)",
      "url": "https://www.unisa.ac.za/",
      "final_url": "https://www.unisa.ac.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:25.360Z"
    },
    {
      "source_id": "uj",
      "name": "University of Johannesburg (UJ)",
      "url": "https://www.uj.ac.za/",
      "final_url": "https://www.uj.ac.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:28.350Z"
    },
    {
      "source_id": "wits",
      "name": "University of the Witwatersrand (Wits)",
      "url": "https://www.wits.ac.za/",
      "final_url": "https://www.wits.ac.za/",
      "status": null,
      "response_time": 8003,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:28.353Z"
    },
    {
      "source_id": "up",
      "name": "University of Pretoria (UP)",
      "url": "https://www.up.ac.za/",
      "final_url": "https://www.up.ac.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:28.355Z"
    },
    {
      "source_id": "uct",
      "name": "University of Cape Town (UCT)",
      "url": "https://www.uct.ac.za/",
      "final_url": "https://www.uct.ac.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:28.358Z"
    },
    {
      "source_id": "ukzn",
      "name": "University of KwaZulu-Natal (UKZN)",
      "url": "https://ukzn.ac.za/",
      "final_url": "https://ukzn.ac.za/",
      "status": null,
      "response_time": 8003,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:28.363Z"
    },
    {
      "source_id": "nmu",
      "name": "Nelson Mandela University (NMU)",
      "url": "https://www.mandela.ac.za/",
      "final_url": "https://www.mandela.ac.za/",
      "status": null,
      "response_time": 8003,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:28.365Z"
    },
    {
      "source_id": "dut",
      "name": "Durban University of Technology (DUT)",
      "url": "https://www.dut.ac.za/",
      "final_url": "https://www.dut.ac.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:28.367Z"
    },
    {
      "source_id": "cut",
      "name": "Central University of Technology (CUT)",
      "url": "https://www.cut.ac.za/",
      "final_url": "https://www.cut.ac.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:33.362Z"
    },
    {
      "source_id": "tut",
      "name": "Tshwane University of Technology (TUT)",
      "url": "https://www.tut.ac.za/",
      "final_url": "https://www.tut.ac.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:36.352Z"
    },
    {
      "source_id": "eskom",
      "name": "Eskom",
      "url": "https://www.eskom.co.za/",
      "final_url": "https://www.eskom.co.za/",
      "status": null,
      "response_time": 8003,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:36.356Z"
    },
    {
      "source_id": "transnet",
      "name": "Transnet",
      "url": "https://www.transnet.net/",
      "final_url": "https://www.transnet.net/",
      "status": null,
      "response_time": 8003,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:36.358Z"
    },
    {
      "source_id": "prasa",
      "name": "Passenger Rail Agency of South Africa (PRASA)",
      "url": "https://www.prasa.com/",
      "final_url": "https://www.prasa.com/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:36.359Z"
    },
    {
      "source_id": "sanral",
      "name": "South African National Roads Agency (SANRAL)",
      "url": "https://www.sanral.co.za/",
      "final_url": "https://www.sanral.co.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:36.364Z"
    },
    {
      "source_id": "saa",
      "name": "South African Airways (SAA)",
      "url": "https://www.flysaa.com/",
      "final_url": "https://www.flysaa.com/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:36.366Z"
    },
    {
      "source_id": "denel",
      "name": "Denel",
      "url": "https://www.denel.co.za/",
      "final_url": "https://www.denel.co.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:36.368Z"
    },
    {
      "source_id": "standard-bank",
      "name": "Standard Bank",
      "url": "https://www.standardbank.com/sbg/standard-bank-group/careers",
      "final_url": "https://www.standardbank.com/sbg/standard-bank-group/careers",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:41.364Z"
    },
    {
      "source_id": "fnb",
      "name": "First National Bank (FNB)",
      "url": "https://www.fnb.co.za/",
      "final_url": "https://www.fnb.co.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:44.354Z"
    },
    {
      "source_id": "absa",
      "name": "Absa",
      "url": "https://www.absa.africa/absaafrica/careers/",
      "final_url": "https://www.absa.africa/absaafrica/careers/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:44.358Z"
    },
    {
      "source_id": "nedbank",
      "name": "Nedbank",
      "url": "https://www.nedbank.co.za/content/nedbank/desktop/gt/en/careers.html",
      "final_url": "https://www.nedbank.co.za/content/nedbank/desktop/gt/en/careers.html",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:44.359Z"
    },
    {
      "source_id": "mtn",
      "name": "MTN Group",
      "url": "https://group.mtn.com/careers/",
      "final_url": "https://group.mtn.com/careers/",
      "status": null,
      "response_time": 8003,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:44.362Z"
    },
    {
      "source_id": "vodacom",
      "name": "Vodacom",
      "url": "https://www.vodacom.com/careers.php",
      "final_url": "https://www.vodacom.com/careers.php",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:44.365Z"
    },
    {
      "source_id": "sasol",
      "name": "Sasol",
      "url": "https://www.sasol.com/careers",
      "final_url": "https://www.sasol.com/careers",
      "status": null,
      "response_time": 8000,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:44.366Z"
    },
    {
      "source_id": "shoprite",
      "name": "Shoprite Group",
      "url": "https://www.shopriteholdings.co.za/careers.html",
      "final_url": "https://www.shopriteholdings.co.za/careers.html",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:44.369Z"
    },
    {
      "source_id": "pick-n-pay",
      "name": "Pick n Pay",
      "url": "https://www.pnp.co.za/careers",
      "final_url": "https://www.pnp.co.za/careers",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:49.366Z"
    },
    {
      "source_id": "woolworths",
      "name": "Woolworths South Africa",
      "url": "https://www.woolworths.co.za/corporate/careers",
      "final_url": "https://www.woolworths.co.za/corporate/careers",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:52.356Z"
    },
    {
      "source_id": "city-of-johannesburg",
      "name": "City of Johannesburg",
      "url": "https://www.joburg.org.za/",
      "final_url": "https://www.joburg.org.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:52.360Z"
    },
    {
      "source_id": "city-of-cape-town",
      "name": "City of Cape Town",
      "url": "https://www.capetown.gov.za/",
      "final_url": "https://www.capetown.gov.za/",
      "status": null,
      "response_time": 8003,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:52.362Z"
    },
    {
      "source_id": "ethekwini",
      "name": "eThekwini Municipality",
      "url": "https://www.durban.gov.za/",
      "final_url": "https://www.durban.gov.za/",
      "status": null,
      "response_time": 8003,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:52.365Z"
    },
    {
      "source_id": "tshwane",
      "name": "City of Tshwane",
      "url": "https://www.tshwane.gov.za/",
      "final_url": "https://www.tshwane.gov.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:52.366Z"
    },
    {
      "source_id": "ekurhuleni",
      "name": "City of Ekurhuleni",
      "url": "https://www.ekurhuleni.gov.za/",
      "final_url": "https://www.ekurhuleni.gov.za/",
      "status": null,
      "response_time": 8003,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:52.369Z"
    },
    {
      "source_id": "nelson-mandela-bay",
      "name": "Nelson Mandela Bay Municipality",
      "url": "https://www.nelsonmandelabay.gov.za/",
      "final_url": "https://www.nelsonmandelabay.gov.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:52.370Z"
    },
    {
      "source_id": "mangaung",
      "name": "Mangaung Metropolitan Municipality",
      "url": "https://www.mangaung.co.za/",
      "final_url": "https://www.mangaung.co.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:21:57.367Z"
    },
    {
      "source_id": "buffalo-city",
      "name": "Buffalo City Metropolitan Municipality",
      "url": "https://www.buffalocity.gov.za/",
      "final_url": "https://www.buffalocity.gov.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:22:00.358Z"
    },
    {
      "source_id": "msunduzi",
      "name": "Msunduzi Municipality",
      "url": "https://www.msunduzi.gov.za/",
      "final_url": "https://www.msunduzi.gov.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:22:00.362Z"
    },
    {
      "source_id": "stellenbosch-municipality",
      "name": "Stellenbosch Municipality",
      "url": "https://stellenbosch.gov.za/",
      "final_url": "https://stellenbosch.gov.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:22:00.364Z"
    },
    {
      "source_id": "western-cape-government",
      "name": "Western Cape Government",
      "url": "https://www.westerncape.gov.za/jobs",
      "final_url": "https://www.westerncape.gov.za/jobs",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:22:00.366Z"
    },
    {
      "source_id": "gauteng-government",
      "name": "Gauteng Provincial Government",
      "url": "https://www.gauteng.gov.za/",
      "final_url": "https://www.gauteng.gov.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:22:00.367Z"
    },
    {
      "source_id": "kzn-government",
      "name": "KwaZulu-Natal Provincial Government",
      "url": "https://www.kznonline.gov.za/",
      "final_url": "https://www.kznonline.gov.za/",
      "status": null,
      "response_time": 8001,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:22:00.370Z"
    },
    {
      "source_id": "csir",
      "name": "Council for Scientific and Industrial Research (CSIR)",
      "url": "https://www.csir.co.za/careers",
      "final_url": "https://www.csir.co.za/careers",
      "status": null,
      "response_time": 8000,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:22:00.370Z"
    },
    {
      "source_id": "nsfas",
      "name": "National Student Financial Aid Scheme (NSFAS)",
      "url": "https://www.nsfas.org.za/",
      "final_url": "https://www.nsfas.org.za/",
      "status": null,
      "response_time": 8002,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:22:05.369Z"
    },
    {
      "source_id": "dhet",
      "name": "Department of Higher Education and Training",
      "url": "https://www.dhet.gov.za/",
      "final_url": "https://www.dhet.gov.za/",
      "status": null,
      "response_time": 8003,
      "redirects": 0,
      "redirect_chain": [],
      "ssl_available": false,
      "healthy": false,
      "error": "Timed out after 8000ms",
      "checked_at": "2026-06-03T20:22:08.361Z"
    }
  ]
}
````

## File: scripts/validators/source-validator.js
````javascript
const fs = require('fs');
const path = require('path');

const SOURCE_TYPES = Object.freeze([
  'government',
  'seta',
  'university',
  'municipality',
  'soe',
  'private',
]);

const CRAWL_STRATEGIES = Object.freeze(['html', 'pdf', 'rss', 'api', 'hybrid']);
const FREQUENCIES = Object.freeze(['hourly', 'daily', 'weekly', 'monthly']);
const PRIORITIES = Object.freeze([1, 2, 3, 4]);

const REQUIRED_FIELDS = Object.freeze([
  'source_id',
  'name',
  'url',
  'type',
  'active',
  'crawl_strategy',
  'frequency',
  'priority',
  'last_checked',
  'last_success',
  'notes',
]);

const DEFAULT_REGISTRY_PATH = path.resolve(__dirname, '..', '..', 'data', 'sources.json');
const SOURCE_ID_PATTERN = /^[a-z0-9]+(?:-[a-z0-9]+)*$/;

function createIssue(level, code, message, source = null, field = null) {
  return { level, code, message, source_id: source ? source.source_id || null : null, field };
}

function isPlainObject(value) {
  return value !== null && typeof value === 'object' && !Array.isArray(value);
}

function isIsoDateOrNull(value) {
  if (value === null) return true;
  if (typeof value !== 'string') return false;

  const timestamp = Date.parse(value);
  return !Number.isNaN(timestamp) && new Date(timestamp).toISOString() === value;
}

function normalizeUrl(value) {
  const parsedUrl = new URL(value);
  parsedUrl.hash = '';

  if ((parsedUrl.protocol === 'https:' && parsedUrl.port === '443') || (parsedUrl.protocol === 'http:' && parsedUrl.port === '80')) {
    parsedUrl.port = '';
  }

  parsedUrl.hostname = parsedUrl.hostname.toLowerCase();
  return parsedUrl.toString().replace(/\/$/, '');
}

function validateUrl(value) {
  if (typeof value !== 'string' || value.trim() === '') {
    return { valid: false, normalizedUrl: null, message: 'url must be a non-empty string.' };
  }

  try {
    const parsedUrl = new URL(value);
    if (!['http:', 'https:'].includes(parsedUrl.protocol)) {
      return { valid: false, normalizedUrl: null, message: 'url must use http or https.' };
    }

    return { valid: true, normalizedUrl: normalizeUrl(value), message: null };
  } catch {
    return { valid: false, normalizedUrl: null, message: 'url must be a valid absolute URL.' };
  }
}

function validateSource(source, index = 0) {
  const issues = [];
  const prefix = `sources[${index}]`;

  if (!isPlainObject(source)) {
    issues.push(createIssue('FAIL', 'INVALID_SOURCE_OBJECT', `${prefix} must be an object.`));
    return issues;
  }

  for (const field of REQUIRED_FIELDS) {
    if (!Object.prototype.hasOwnProperty.call(source, field)) {
      issues.push(createIssue('FAIL', 'MISSING_FIELD', `${prefix} is missing required field "${field}".`, source, field));
    }
  }

  if (typeof source.source_id !== 'string' || !SOURCE_ID_PATTERN.test(source.source_id)) {
    issues.push(createIssue('FAIL', 'INVALID_SOURCE_ID', `${prefix}.source_id must be a non-empty kebab-case identifier.`, source, 'source_id'));
  }

  if (typeof source.name !== 'string' || source.name.trim() === '') {
    issues.push(createIssue('FAIL', 'INVALID_NAME', `${prefix}.name must be a non-empty string.`, source, 'name'));
  }

  if (!SOURCE_TYPES.includes(source.type)) {
    issues.push(createIssue('FAIL', 'INVALID_SOURCE_TYPE', `${prefix}.type must be one of: ${SOURCE_TYPES.join(', ')}.`, source, 'type'));
  }

  const urlResult = validateUrl(source.url);
  if (!urlResult.valid) {
    issues.push(createIssue('FAIL', 'INVALID_URL', `${prefix}.${urlResult.message}`, source, 'url'));
  }

  if (typeof source.active !== 'boolean') {
    issues.push(createIssue('FAIL', 'INVALID_ACTIVE', `${prefix}.active must be a boolean.`, source, 'active'));
  }

  if (!CRAWL_STRATEGIES.includes(source.crawl_strategy)) {
    issues.push(createIssue('FAIL', 'INVALID_CRAWL_STRATEGY', `${prefix}.crawl_strategy must be one of: ${CRAWL_STRATEGIES.join(', ')}.`, source, 'crawl_strategy'));
  }

  if (!FREQUENCIES.includes(source.frequency)) {
    issues.push(createIssue('FAIL', 'INVALID_FREQUENCY', `${prefix}.frequency must be one of: ${FREQUENCIES.join(', ')}.`, source, 'frequency'));
  }

  if (!PRIORITIES.includes(source.priority)) {
    issues.push(createIssue('FAIL', 'INVALID_PRIORITY', `${prefix}.priority must be one of: ${PRIORITIES.join(', ')}.`, source, 'priority'));
  }

  if (!isIsoDateOrNull(source.last_checked)) {
    issues.push(createIssue('FAIL', 'INVALID_LAST_CHECKED', `${prefix}.last_checked must be null or an ISO-8601 UTC timestamp.`, source, 'last_checked'));
  }

  if (!isIsoDateOrNull(source.last_success)) {
    issues.push(createIssue('FAIL', 'INVALID_LAST_SUCCESS', `${prefix}.last_success must be null or an ISO-8601 UTC timestamp.`, source, 'last_success'));
  }

  if (typeof source.notes !== 'string') {
    issues.push(createIssue('FAIL', 'INVALID_NOTES', `${prefix}.notes must be a string.`, source, 'notes'));
  } else if (source.notes.trim() === '') {
    issues.push(createIssue('WARN', 'EMPTY_NOTES', `${prefix}.notes is empty; add source context for operators.`, source, 'notes'));
  }

  return issues;
}

function validateDuplicates(sources) {
  const issues = [];
  const idMap = new Map();
  const urlMap = new Map();

  sources.forEach((source, index) => {
    if (!isPlainObject(source)) return;

    if (typeof source.source_id === 'string' && source.source_id.trim() !== '') {
      const existingIndex = idMap.get(source.source_id);
      if (existingIndex !== undefined) {
        issues.push(createIssue('FAIL', 'DUPLICATE_SOURCE_ID', `sources[${index}].source_id duplicates sources[${existingIndex}].source_id: "${source.source_id}".`, source, 'source_id'));
      } else {
        idMap.set(source.source_id, index);
      }
    }

    const urlResult = validateUrl(source.url);
    if (urlResult.valid) {
      const existingIndex = urlMap.get(urlResult.normalizedUrl);
      if (existingIndex !== undefined) {
        issues.push(createIssue('FAIL', 'DUPLICATE_URL', `sources[${index}].url duplicates sources[${existingIndex}].url: "${urlResult.normalizedUrl}".`, source, 'url'));
      } else {
        urlMap.set(urlResult.normalizedUrl, index);
      }
    }
  });

  return issues;
}

function validateRegistry(sources, options = {}) {
  const issues = [];
  const minimumSources = options.minimumSources ?? 1;

  if (!Array.isArray(sources)) {
    return {
      valid: false,
      status: 'FAIL',
      issues: [createIssue('FAIL', 'INVALID_REGISTRY_ROOT', 'Source registry root must be an array.')],
      summary: { total: 0, active: 0, inactive: 0, by_type: {}, issue_counts: { PASS: 0, WARN: 0, FAIL: 1 } },
    };
  }

  if (sources.length < minimumSources) {
    issues.push(createIssue('FAIL', 'MINIMUM_SOURCE_COUNT', `Source registry must contain at least ${minimumSources} sources; found ${sources.length}.`));
  }

  sources.forEach((source, index) => {
    issues.push(...validateSource(source, index));
  });
  issues.push(...validateDuplicates(sources));

  const byType = sources.reduce((summary, source) => {
    const type = isPlainObject(source) ? source.type || 'unknown' : 'unknown';
    summary[type] = (summary[type] || 0) + 1;
    return summary;
  }, {});

  const active = sources.filter((source) => isPlainObject(source) && source.active === true).length;
  const issueCounts = issues.reduce((counts, issue) => {
    counts[issue.level] = (counts[issue.level] || 0) + 1;
    return counts;
  }, { PASS: 0, WARN: 0, FAIL: 0 });

  const status = issueCounts.FAIL > 0 ? 'FAIL' : issueCounts.WARN > 0 ? 'WARN' : 'PASS';

  return {
    valid: issueCounts.FAIL === 0,
    status,
    issues,
    summary: {
      total: sources.length,
      active,
      inactive: sources.length - active,
      by_type: byType,
      issue_counts: issueCounts,
    },
  };
}

function loadSources(filePath = DEFAULT_REGISTRY_PATH) {
  const raw = fs.readFileSync(filePath, 'utf8');
  return JSON.parse(raw);
}

module.exports = {
  CRAWL_STRATEGIES,
  DEFAULT_REGISTRY_PATH,
  FREQUENCIES,
  PRIORITIES,
  REQUIRED_FIELDS,
  SOURCE_TYPES,
  isIsoDateOrNull,
  loadSources,
  normalizeUrl,
  validateDuplicates,
  validateRegistry,
  validateSource,
  validateUrl,
};
````

## File: scripts/check-links.js
````javascript
#!/usr/bin/env node
/**
 * Checks canonical opportunity source and application URLs.
 *
 * The checker fails on hard-broken HTTP responses such as 404/410 while
 * treating bot-blocking or transient network failures as warnings by default.
 * Set LINK_CHECK_STRICT=true to fail on warnings in scheduled maintenance runs.
 */

const fs = require('fs');
const path = require('path');
const { execFile } = require('child_process');

const ROOT = path.resolve(__dirname, '..');
const DATA_PATH = path.join(ROOT, 'data', 'opportunities.json');
const TIMEOUT_MS = Number(process.env.LINK_CHECK_TIMEOUT_MS || 8000);
const RETRIES = Number(process.env.LINK_CHECK_RETRIES || 2);
const CONCURRENCY = Number(process.env.LINK_CHECK_CONCURRENCY || 4);
const STRICT = process.env.LINK_CHECK_STRICT === 'true';
const HARD_BROKEN_STATUSES = new Set([404, 410, 451]);
const WARNING_STATUSES = new Set([401, 403, 429]);

const opportunities = JSON.parse(fs.readFileSync(DATA_PATH, 'utf8'));
const urlMap = new Map();

for (const opportunity of opportunities) {
  for (const field of ['application_url', 'source_url']) {
    const value = opportunity[field];
    if (!urlMap.has(value)) urlMap.set(value, []);
    urlMap.get(value).push(`${opportunity.id}:${field}`);
  }
}

function classifyStatus(status) {
  if (status >= 200 && status < 400) return { level: 'pass', message: `HTTP ${status}` };
  if (HARD_BROKEN_STATUSES.has(status)) return { level: 'error', message: `HTTP ${status}` };
  if (WARNING_STATUSES.has(status)) return { level: 'warning', message: `HTTP ${status} (reachable but restricted or rate-limited)` };
  if (status >= 500) return { level: 'warning', message: `HTTP ${status} (server-side response)` };
  return { level: 'error', message: `HTTP ${status}` };
}

function mergeLevels(left, right) {
  const rank = { pass: 0, warning: 1, error: 2 };
  return rank[right] > rank[left] ? right : left;
}

function request(url, method) {
  const timeoutSeconds = Math.ceil(TIMEOUT_MS / 1000);
  const args = [
    '--location',
    '--silent',
    '--show-error',
    '--max-time', String(timeoutSeconds),
    '--output', '/dev/null',
    '--write-out', '%{http_code}',
    '--user-agent', 'OpportunitiesZA-LinkChecker/1.0 (+https://opportunitiesza.co.za)',
  ];

  if (method === 'HEAD') args.push('--head');
  args.push(url);

  return new Promise((resolve, reject) => {
    execFile('curl', args, { timeout: TIMEOUT_MS + 1000 }, (error, stdout, stderr) => {
      const status = Number(String(stdout).trim().slice(-3));
      if (status) {
        resolve({ status });
        return;
      }

      if (error) {
        reject(new Error((stderr || error.message).trim()));
        return;
      }

      reject(new Error('curl did not return an HTTP status code'));
    });
  });
}

async function checkUrl(url, refs) {
  let parsed;
  try {
    parsed = new URL(url);
  } catch (error) {
    return { url, refs, level: 'error', message: `Invalid URL: ${error.message}` };
  }

  if (parsed.protocol !== 'https:' && parsed.protocol !== 'http:') {
    return { url, refs, level: 'error', message: `Unsupported protocol: ${parsed.protocol}` };
  }

  let lastError;
  for (let attempt = 1; attempt <= RETRIES + 1; attempt += 1) {
    try {
      let response = await request(url, 'HEAD');
      if ([405, 501].includes(response.status)) response = await request(url, 'GET');
      const statusResult = classifyStatus(response.status);
      return { url, refs, ...statusResult };
    } catch (error) {
      lastError = error;
      if (attempt <= RETRIES) {
        await new Promise(resolve => setTimeout(resolve, 350 * attempt));
      }
    }
  }

  return {
    url,
    refs,
    level: STRICT ? 'error' : 'warning',
    message: `Network check failed after ${RETRIES + 1} attempt(s): ${lastError.message}`,
  };
}

async function runQueue(items) {
  const results = [];
  let cursor = 0;

  async function worker() {
    while (cursor < items.length) {
      const index = cursor;
      cursor += 1;
      const [url, refs] = items[index];
      results[index] = await checkUrl(url, refs);
    }
  }

  await Promise.all(Array.from({ length: Math.min(CONCURRENCY, items.length) }, worker));
  return results;
}

async function main() {
  const results = await runQueue([...urlMap.entries()]);
  let overall = 'pass';

  for (const result of results) {
    overall = mergeLevels(overall, result.level);
    const icon = result.level === 'pass' ? '✅' : result.level === 'warning' ? '⚠️' : '❌';
    console.log(`${icon} ${result.url} — ${result.message} (${result.refs.join(', ')})`);
  }

  const warnings = results.filter(result => result.level === 'warning').length;
  const errors = results.filter(result => result.level === 'error').length;
  console.log(`\nChecked ${results.length} unique opportunity URL(s): ${errors} error(s), ${warnings} warning(s).`);

  if (errors > 0 || (STRICT && warnings > 0)) process.exit(1);
}

main();
````

## File: scripts/normalize_dataset.js
````javascript
const fs = require("fs");
const path = require("path");

const INPUT_PATHS = {
  opportunities: "./data/opportunities.json",
  guides: "./data/guides.json",
  categories: "./data/categories.json",
  provinces: "./data/provinces.json"
};

const OUTPUT_PATH = "./data/clean/";
const LOG_PATH = "./logs/";

const DATE_FIELDS = ["closing_date", "posted_date", "updated_date", "published_date"];
const URL_FIELDS = ["source_url", "application_url"];

const PROVINCE_MAP = {
  gp: "Gauteng",
  gauteng: "Gauteng",
  "gauteng province": "Gauteng",

  wc: "Western Cape",
  "western cape": "Western Cape",
  "western-cape": "Western Cape",

  kzn: "KwaZulu-Natal",
  "kwazulu natal": "KwaZulu-Natal",
  "kwazulu-natal": "KwaZulu-Natal"
};

const CATEGORY_MAP = {
  learnerships: "learnership",
  learnership: "learnership",

  internships: "internship",
  internship: "internship",

  bursaries: "bursary",
  bursary: "bursary",

  apprenticeships: "apprenticeship",
  apprenticeship: "apprenticeship"
};

function ensureBackupExists() {
  if (!fs.existsSync("./data/backup")) {
    fs.mkdirSync("./data/backup", { recursive: true });
  }
}

function ensureOutputDirectories() {
  fs.mkdirSync(OUTPUT_PATH, { recursive: true });
  fs.mkdirSync(LOG_PATH, { recursive: true });
}

function resolveInputPath(inputPath) {
  if (fs.existsSync(inputPath)) return inputPath;

  const alternatePath = inputPath.replace(/^\.\/data\//, "./Data/");
  if (fs.existsSync(alternatePath)) return alternatePath;

  return inputPath;
}

function loadJSON(inputPath) {
  const resolvedPath = resolveInputPath(inputPath);

  try {
    const raw = fs.readFileSync(resolvedPath, "utf8");
    return JSON.parse(raw);
  } catch (err) {
    console.error(`Invalid JSON: ${resolvedPath}`, err.message);
    return null;
  }
}

function normalizeProvince(value) {
  if (!value) return null;

  const key = value.toString().trim().toLowerCase();

  return PROVINCE_MAP[key] || value;
}

function normalizeCategory(value) {
  if (!value) return null;

  const key = value.toString().trim().toLowerCase();

  return CATEGORY_MAP[key] || value;
}

function normalizeBoolean(value) {
  if (value === true || value === false) return value;

  if (typeof value === "string") {
    const v = value.trim().toLowerCase();
    if (v === "true" || v === "yes" || v === "1") return true;
    if (v === "false" || v === "no" || v === "0") return false;
  }

  if (value === 1) return true;
  if (value === 0) return false;

  return null;
}

function isValidISODateParts(year, month, day) {
  const date = new Date(Date.UTC(year, month - 1, day));

  return (
    date.getUTCFullYear() === year &&
    date.getUTCMonth() === month - 1 &&
    date.getUTCDate() === day
  );
}

function normalizeDate(value) {
  if (!value) return null;

  const stringValue = value.toString().trim();
  const isoDateMatch = stringValue.match(/^(\d{4})-(\d{2})-(\d{2})$/);

  if (isoDateMatch) {
    const [, year, month, day] = isoDateMatch.map(Number);

    if (!isValidISODateParts(year, month, day)) {
      return { error: "INVALID_DATE", value };
    }

    return stringValue;
  }

  const date = new Date(stringValue);

  if (isNaN(date.getTime())) {
    return { error: "INVALID_DATE", value };
  }

  return date.toISOString().split("T")[0];
}

function normalizeURL(url) {
  if (!url) return null;

  let cleaned = url.toString().trim();

  if (!/^https?:\/\//i.test(cleaned)) {
    cleaned = "https://" + cleaned;
  }

  return cleaned;
}

function normalizeOpportunity(record) {
  const changes = [];

  const normalized = { ...record };

  // Province
  if (record.province !== undefined) {
    const oldProvince = record.province;
    const newProvince = normalizeProvince(oldProvince);

    if (oldProvince !== newProvince) {
      changes.push({
        field: "province",
        from: oldProvince,
        to: newProvince
      });
      normalized.province = newProvince;
    }
  }

  // Category
  if (record.category !== undefined) {
    const oldCategory = record.category;
    const newCategory = normalizeCategory(oldCategory);

    if (oldCategory !== newCategory) {
      changes.push({
        field: "category",
        from: oldCategory,
        to: newCategory
      });
      normalized.category = newCategory;
    }
  }

  // Boolean fields
  if (record.verified !== undefined) {
    const oldVal = record.verified;
    const newVal = normalizeBoolean(oldVal);

    if (oldVal !== newVal) {
      changes.push({
        field: "verified",
        from: oldVal,
        to: newVal
      });
      normalized.verified = newVal;
    }
  }

  // Dates
  DATE_FIELDS.forEach(field => {
    if (record[field]) {
      const oldVal = record[field];
      const newVal = normalizeDate(oldVal);

      if (typeof newVal === "object" && newVal.error) {
        changes.push({
          field,
          error: "INVALID_DATE",
          value: oldVal
        });
      } else if (oldVal !== newVal) {
        changes.push({
          field,
          from: oldVal,
          to: newVal
        });
        normalized[field] = newVal;
      }
    }
  });

  // URLs
  URL_FIELDS.forEach(field => {
    if (record[field]) {
      const oldVal = record[field];
      const newVal = normalizeURL(oldVal);

      if (oldVal !== newVal) {
        changes.push({
          field,
          from: oldVal,
          to: newVal
        });
        normalized[field] = newVal;
      }
    }
  });

  return { normalized, changes };
}

function processFile(fileKey, data) {
  const results = [];
  const allChanges = [];

  if (!Array.isArray(data)) {
    return {
      results,
      allChanges: [
        {
          id: null,
          changes: [
            {
              file: fileKey,
              error: "INVALID_DATASET_SHAPE",
              value: data === null ? "null" : typeof data
            }
          ]
        }
      ]
    };
  }

  data.forEach(record => {
    const { normalized, changes } = normalizeOpportunity(record);

    results.push(normalized);

    if (changes.length > 0) {
      allChanges.push({
        id: record.id || null,
        changes
      });
    }
  });

  return { results, allChanges };
}

function writeOutput(fileName, data) {
  const outputFile = path.join(OUTPUT_PATH, fileName);
  fs.writeFileSync(outputFile, JSON.stringify(data, null, 2) + "\n");
}

function writeLogs(allChanges) {
  const logFile = path.join(LOG_PATH, "changes-log.json");

  fs.writeFileSync(logFile, JSON.stringify(allChanges, null, 2) + "\n");
}

function buildNormalizationReport(globalChanges, processedFiles) {
  const files = globalChanges.map(fileLog => {
    const modifiedRecords = fileLog.changes.length;
    const totalChanges = fileLog.changes.reduce((sum, recordLog) => sum + recordLog.changes.length, 0);
    const invalidDates = fileLog.changes.reduce(
      (sum, recordLog) => sum + recordLog.changes.filter(change => change.error === "INVALID_DATE").length,
      0
    );

    return {
      file: fileLog.file,
      input: processedFiles[fileLog.file]?.input || INPUT_PATHS[fileLog.file],
      output: `${OUTPUT_PATH}${fileLog.file}.json`,
      recordsProcessed: processedFiles[fileLog.file]?.recordsProcessed || 0,
      modifiedRecords,
      totalChanges,
      invalidDates,
      outputWritten: true
    };
  });

  return {
    generatedAt: new Date().toISOString(),
    safety: {
      originalDataOverwritten: false,
      backupDirectoryEnsured: "./data/backup",
      outputDirectory: OUTPUT_PATH,
      logDirectory: LOG_PATH
    },
    totals: {
      filesProcessed: files.length,
      recordsProcessed: files.reduce((sum, file) => sum + file.recordsProcessed, 0),
      modifiedRecords: files.reduce((sum, file) => sum + file.modifiedRecords, 0),
      totalChanges: files.reduce((sum, file) => sum + file.totalChanges, 0),
      invalidDates: files.reduce((sum, file) => sum + file.invalidDates, 0)
    },
    files
  };
}

function writeNormalizationReport(report) {
  const reportFile = path.join(LOG_PATH, "normalization-report.json");
  fs.writeFileSync(reportFile, JSON.stringify(report, null, 2) + "\n");
}

function run() {
  ensureBackupExists();
  ensureOutputDirectories();

  const globalChanges = [];
  const processedFiles = {};

  Object.keys(INPUT_PATHS).forEach(key => {
    const data = loadJSON(INPUT_PATHS[key]);

    if (!data) return;

    const { results, allChanges } = processFile(key, data);

    writeOutput(key + ".json", results);

    processedFiles[key] = {
      input: resolveInputPath(INPUT_PATHS[key]),
      recordsProcessed: Array.isArray(data) ? data.length : 0
    };

    globalChanges.push({
      file: key,
      changes: allChanges
    });
  });

  writeLogs(globalChanges);
  writeNormalizationReport(buildNormalizationReport(globalChanges, processedFiles));

  console.log("Normalization complete.");
}

if (require.main === module) {
  run();
}

module.exports = {
  CATEGORY_MAP,
  PROVINCE_MAP,
  normalizeBoolean,
  normalizeCategory,
  normalizeDate,
  normalizeOpportunity,
  normalizeProvince,
  normalizeURL,
  processFile,
  run
};
````

## File: scripts/source-audit.js
````javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');
const {
  DEFAULT_REGISTRY_PATH,
  SOURCE_TYPES,
  loadSources,
  validateRegistry,
} = require('./validators/source-validator');

const REPORTS_DIR = path.resolve(__dirname, '..', 'reports');
const REPORT_PATH = path.join(REPORTS_DIR, 'source-audit-report.json');
const MINIMUM_SOURCE_COUNT = 50;

function ensureReportsDir() {
  fs.mkdirSync(REPORTS_DIR, { recursive: true });
}

function buildReport(validation, registryPath = DEFAULT_REGISTRY_PATH) {
  return {
    generated_at: new Date().toISOString(),
    registry_path: path.relative(path.resolve(__dirname, '..'), registryPath),
    status: validation.status,
    summary: validation.summary,
    issues: validation.issues,
  };
}

function writeReport(report, reportPath = REPORT_PATH) {
  ensureReportsDir();
  fs.writeFileSync(reportPath, `${JSON.stringify(report, null, 2)}\n`);
}

function printAudit(report) {
  console.log(`Source audit: ${report.status}`);
  console.log(`Sources: ${report.summary.total} total, ${report.summary.active} active, ${report.summary.inactive} inactive.`);
  console.log(`Issues: ${report.summary.issue_counts.FAIL} FAIL, ${report.summary.issue_counts.WARN} WARN.`);
  console.log('Sources by type:');

  for (const type of SOURCE_TYPES) {
    console.log(`- ${type}: ${report.summary.by_type[type] || 0}`);
  }

  if (report.issues.length > 0) {
    console.log('\nFindings:');
    for (const issue of report.issues) {
      console.log(`[${issue.level}] ${issue.code}: ${issue.message}`);
    }
  }

  console.log(`\nReport written to ${path.relative(process.cwd(), REPORT_PATH)}`);
}

function runAudit(options = {}) {
  const registryPath = options.registryPath || DEFAULT_REGISTRY_PATH;
  const sources = loadSources(registryPath);
  const validation = validateRegistry(sources, { minimumSources: options.minimumSources ?? MINIMUM_SOURCE_COUNT });
  const report = buildReport(validation, registryPath);
  writeReport(report, options.reportPath || REPORT_PATH);
  return report;
}

function main() {
  let report;

  try {
    report = runAudit();
  } catch (error) {
    const failure = {
      generated_at: new Date().toISOString(),
      registry_path: path.relative(path.resolve(__dirname, '..'), DEFAULT_REGISTRY_PATH),
      status: 'FAIL',
      summary: { total: 0, active: 0, inactive: 0, by_type: {}, issue_counts: { PASS: 0, WARN: 0, FAIL: 1 } },
      issues: [{ level: 'FAIL', code: 'AUDIT_EXCEPTION', message: error.message, source_id: null, field: null }],
    };
    writeReport(failure);
    printAudit(failure);
    process.exit(1);
  }

  printAudit(report);

  if (report.status === 'FAIL') {
    process.exit(1);
  }
}

if (require.main === module) {
  main();
}

module.exports = {
  MINIMUM_SOURCE_COUNT,
  REPORT_PATH,
  REPORTS_DIR,
  buildReport,
  runAudit,
  writeReport,
};
````

## File: scripts/source-health-check.js
````javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');
const tls = require('tls');
const { URL } = require('url');
const { DEFAULT_REGISTRY_PATH, loadSources, validateRegistry } = require('./validators/source-validator');

const REPORTS_DIR = path.resolve(__dirname, '..', 'reports');
const REPORT_PATH = path.join(REPORTS_DIR, 'source-health-report.json');
const DEFAULT_TIMEOUT_MS = 8000;
const DEFAULT_CONCURRENCY = 8;
const MAX_REDIRECTS = 5;

function getConfig() {
  return {
    timeoutMs: Number.parseInt(process.env.SOURCE_HEALTH_TIMEOUT_MS || `${DEFAULT_TIMEOUT_MS}`, 10),
    limit: Number.parseInt(process.env.SOURCE_HEALTH_LIMIT || '0', 10),
    strict: process.env.SOURCE_HEALTH_STRICT === '1',
    concurrency: Number.parseInt(process.env.SOURCE_HEALTH_CONCURRENCY || `${DEFAULT_CONCURRENCY}`, 10),
  };
}

function ensureReportsDir() {
  fs.mkdirSync(REPORTS_DIR, { recursive: true });
}

function writeReport(report, reportPath = REPORT_PATH) {
  ensureReportsDir();
  fs.writeFileSync(reportPath, `${JSON.stringify(report, null, 2)}\n`);
}

function checkSslAvailability(sourceUrl, timeoutMs) {
  return new Promise((resolve) => {
    let settled = false;
    let socket;

    function finish(value) {
      if (settled) return;
      settled = true;
      if (socket && !socket.destroyed) {
        socket.destroy();
      }
      resolve(value);
    }
    let parsedUrl;

    try {
      parsedUrl = new URL(sourceUrl);
    } catch {
      finish(false);
      return;
    }

    if (parsedUrl.protocol !== 'https:') {
      finish(false);
      return;
    }

    socket = tls.connect({
      host: parsedUrl.hostname,
      port: parsedUrl.port ? Number.parseInt(parsedUrl.port, 10) : 443,
      servername: parsedUrl.hostname,
      timeout: timeoutMs,
    });

    socket.once('secureConnect', () => {
      const authorized = socket.authorized || socket.authorizationError === null;
      finish(Boolean(authorized));
    });

    socket.once('timeout', () => {
      finish(false);
    });

    socket.once('error', () => {
      finish(false);
    });
  });
}

function resolveRedirectUrl(currentUrl, location) {
  return new URL(location, currentUrl).toString();
}

async function fetchWithRedirects(sourceUrl, timeoutMs, method = 'HEAD') {
  let currentUrl = sourceUrl;
  const chain = [];

  for (let redirectCount = 0; redirectCount <= MAX_REDIRECTS; redirectCount += 1) {
    const controller = new AbortController();
    const timeout = setTimeout(() => controller.abort(), timeoutMs);

    try {
      const response = await fetch(currentUrl, {
        method,
        redirect: 'manual',
        signal: controller.signal,
        headers: {
          'User-Agent': 'seta-careers-platform-source-health/1.0',
          Accept: '*/*',
        },
      });

      const location = response.headers.get('location');
      if ([301, 302, 303, 307, 308].includes(response.status) && location) {
        const nextUrl = resolveRedirectUrl(currentUrl, location);
        chain.push({ status: response.status, from: currentUrl, to: nextUrl });
        currentUrl = nextUrl;
        continue;
      }

      return { response, finalUrl: currentUrl, redirectChain: chain };
    } finally {
      clearTimeout(timeout);
    }
  }

  throw new Error(`Exceeded ${MAX_REDIRECTS} redirects.`);
}

async function probeSource(source, config = getConfig()) {
  const startedAt = Date.now();
  const sslAvailablePromise = checkSslAvailability(source.url, config.timeoutMs);

  try {
    let result = await fetchWithRedirects(source.url, config.timeoutMs, 'HEAD');

    if ([403, 405].includes(result.response.status)) {
      result = await fetchWithRedirects(source.url, config.timeoutMs, 'GET');
    }

    const responseTime = Date.now() - startedAt;
    const sslAvailable = await sslAvailablePromise;
    const healthy = result.response.status >= 200 && result.response.status < 400;

    return {
      source_id: source.source_id,
      name: source.name,
      url: source.url,
      final_url: result.finalUrl,
      status: result.response.status,
      response_time: responseTime,
      redirects: result.redirectChain.length,
      redirect_chain: result.redirectChain,
      ssl_available: sslAvailable,
      healthy,
      checked_at: new Date().toISOString(),
    };
  } catch (error) {
    const responseTime = Date.now() - startedAt;
    const sslAvailable = await sslAvailablePromise;

    return {
      source_id: source.source_id,
      name: source.name,
      url: source.url,
      final_url: source.url,
      status: null,
      response_time: responseTime,
      redirects: 0,
      redirect_chain: [],
      ssl_available: sslAvailable,
      healthy: false,
      error: error.name === 'AbortError' ? `Timed out after ${config.timeoutMs}ms` : error.message,
      checked_at: new Date().toISOString(),
    };
  }
}

async function mapWithConcurrency(items, concurrency, mapper) {
  const results = new Array(items.length);
  let nextIndex = 0;

  async function worker() {
    while (nextIndex < items.length) {
      const currentIndex = nextIndex;
      nextIndex += 1;
      results[currentIndex] = await mapper(items[currentIndex], currentIndex);
    }
  }

  const workerCount = Math.min(Math.max(concurrency, 1), items.length || 1);
  await Promise.all(Array.from({ length: workerCount }, worker));
  return results;
}

function buildReport(results, config) {
  const healthyCount = results.filter((result) => result.healthy).length;
  const unhealthyCount = results.length - healthyCount;
  const warnCount = results.filter((result) => result.ssl_available === false || result.redirects > 0).length;

  return {
    generated_at: new Date().toISOString(),
    status: unhealthyCount > 0 ? 'WARN' : warnCount > 0 ? 'WARN' : 'PASS',
    summary: {
      checked: results.length,
      healthy: healthyCount,
      unhealthy: unhealthyCount,
      with_redirects: results.filter((result) => result.redirects > 0).length,
      ssl_available: results.filter((result) => result.ssl_available).length,
      timeout_ms: config.timeoutMs,
    },
    sources: results,
  };
}

function printHealth(report) {
  console.log(`Source health: ${report.status}`);
  console.log(`Checked: ${report.summary.checked}; healthy: ${report.summary.healthy}; unhealthy: ${report.summary.unhealthy}; redirects: ${report.summary.with_redirects}; SSL available: ${report.summary.ssl_available}.`);

  for (const result of report.sources) {
    const level = result.healthy ? (result.redirects > 0 || !result.ssl_available ? 'WARN' : 'PASS') : 'FAIL';
    const status = result.status === null ? 'n/a' : result.status;
    const error = result.error ? ` (${result.error})` : '';
    console.log(`[${level}] ${result.source_id}: status=${status}, response_time=${result.response_time}ms, redirects=${result.redirects}, ssl=${result.ssl_available}${error}`);
  }

  console.log(`\nReport written to ${path.relative(process.cwd(), REPORT_PATH)}`);
}

async function runHealthCheck(options = {}) {
  const config = { ...getConfig(), ...options };
  const sources = loadSources(options.registryPath || DEFAULT_REGISTRY_PATH);
  const validation = validateRegistry(sources, { minimumSources: 1 });

  if (!validation.valid) {
    const report = {
      generated_at: new Date().toISOString(),
      status: 'FAIL',
      summary: { checked: 0, healthy: 0, unhealthy: 0, with_redirects: 0, ssl_available: 0, timeout_ms: config.timeoutMs },
      validation_issues: validation.issues,
      sources: [],
    };
    writeReport(report, options.reportPath || REPORT_PATH);
    return report;
  }

  const activeSources = sources.filter((source) => source.active);
  const selectedSources = config.limit > 0 ? activeSources.slice(0, config.limit) : activeSources;
  const results = await mapWithConcurrency(selectedSources, config.concurrency, (source) => probeSource(source, config));
  const report = buildReport(results, config);
  writeReport(report, options.reportPath || REPORT_PATH);
  return report;
}

async function main() {
  const config = getConfig();
  const report = await runHealthCheck(config);
  printHealth(report);

  if (report.status === 'FAIL' || (config.strict && report.summary.unhealthy > 0)) {
    process.exit(1);
  }

  process.exit(0);
}

if (require.main === module) {
  main().catch((error) => {
    const report = {
      generated_at: new Date().toISOString(),
      status: 'FAIL',
      summary: { checked: 0, healthy: 0, unhealthy: 0, with_redirects: 0, ssl_available: 0, timeout_ms: getConfig().timeoutMs },
      error: error.message,
      sources: [],
    };
    writeReport(report);
    printHealth(report);
    process.exit(1);
  });
}

module.exports = {
  REPORT_PATH,
  REPORTS_DIR,
  buildReport,
  checkSslAvailability,
  probeSource,
  runHealthCheck,
};
````

## File: scripts/validate-content.js
````javascript
#!/usr/bin/env node
/**
 * OpportunitiesZA content quality gate.
 *
 * Validates the JSON data files that power SEO pages, sitemaps, and the
 * editorial workflow. The script intentionally has zero external dependencies
 * so it can run in Termux, GitHub Actions, and local Node installs.
 */

const fs = require('fs');
const path = require('path');

const ROOT = path.resolve(__dirname, '..');
const DATA_DIR = path.join(ROOT, 'data');
const TODAY = new Date().toISOString().slice(0, 10);

const opportunityRequiredFields = [
  'id',
  'slug',
  'title',
  'category',
  'sector',
  'province',
  'qualification_level',
  'stipend',
  'closing_date',
  'application_url',
  'source_name',
  'source_url',
  'verified',
  'featured',
  'expired',
  'description',
  'eligibility',
  'documents_needed',
  'tags',
  'posted_date',
  'updated_date',
];

const guideRequiredFields = [
  'id',
  'slug',
  'title',
  'category',
  'intro',
  'body',
  'faq',
  'meta_title',
  'meta_description',
  'read_time',
  'published_date',
  'updated_date',
];

const categoryIds = new Set(['learnership', 'internship', 'bursary', 'apprenticeship', 'tvet']);
const guideCategories = new Set(['seta-guides', 'career-advice']);
const provinceNames = new Set([
  'Eastern Cape',
  'Free State',
  'Gauteng',
  'KwaZulu-Natal',
  'Limpopo',
  'Mpumalanga',
  'Northern Cape',
  'North West',
  'Western Cape',
  'Nationwide',
]);

const errors = [];
const warnings = [];

function readJson(relativePath) {
  const fullPath = path.join(ROOT, relativePath);
  try {
    return JSON.parse(fs.readFileSync(fullPath, 'utf8'));
  } catch (error) {
    errors.push(`${relativePath}: ${error.message}`);
    return null;
  }
}

function isIsoDate(value) {
  if (typeof value !== 'string' || !/^\d{4}-\d{2}-\d{2}$/.test(value)) return false;
  const date = new Date(`${value}T00:00:00Z`);
  return !Number.isNaN(date.valueOf()) && date.toISOString().slice(0, 10) === value;
}

function isHttpUrl(value) {
  try {
    const url = new URL(value);
    return url.protocol === 'https:' || url.protocol === 'http:';
  } catch {
    return false;
  }
}

function isSlug(value) {
  return typeof value === 'string' && /^[a-z0-9]+(?:-[a-z0-9]+)*$/.test(value);
}

function assertArray(name, value, relativePath) {
  if (!Array.isArray(value)) {
    errors.push(`${relativePath}: expected top-level JSON array`);
    return false;
  }
  return true;
}

function checkRequired(record, fields, label) {
  for (const field of fields) {
    if (!(field in record)) errors.push(`${label}: missing required field "${field}"`);
  }
}

function checkUnique(records, field, label) {
  const seen = new Map();
  for (const record of records) {
    const value = record[field];
    if (seen.has(value)) {
      errors.push(`${label}: duplicate ${field} "${value}" in ${seen.get(value)} and ${record.id || record.slug}`);
    } else {
      seen.set(value, record.id || record.slug);
    }
  }
}

function validateOpportunities(opportunities) {
  if (!assertArray('opportunities', opportunities, 'data/opportunities.json')) return;

  checkUnique(opportunities, 'id', 'opportunities');
  checkUnique(opportunities, 'slug', 'opportunities');

  for (const opportunity of opportunities) {
    const label = `opportunity ${opportunity.id || opportunity.slug || '(unknown)'}`;
    checkRequired(opportunity, opportunityRequiredFields, label);

    if (!isSlug(opportunity.slug)) errors.push(`${label}: slug must be lowercase kebab-case`);
    if (!categoryIds.has(opportunity.category)) errors.push(`${label}: unknown category "${opportunity.category}"`);
    if (!provinceNames.has(opportunity.province)) warnings.push(`${label}: province "${opportunity.province}" is not in the standard province list`);

    for (const field of ['closing_date', 'posted_date', 'updated_date']) {
      if (!isIsoDate(opportunity[field])) errors.push(`${label}: ${field} must be YYYY-MM-DD`);
    }

    if (opportunity.updated_date && opportunity.posted_date && opportunity.updated_date < opportunity.posted_date) {
      errors.push(`${label}: updated_date cannot be before posted_date`);
    }

    if (!opportunity.expired && opportunity.closing_date && opportunity.closing_date < TODAY) {
      warnings.push(`${label}: closing_date ${opportunity.closing_date} is before ${TODAY} but expired is false`);
    }

    for (const field of ['application_url', 'source_url']) {
      if (!isHttpUrl(opportunity[field])) errors.push(`${label}: ${field} must be a valid http(s) URL`);
    }

    for (const field of ['verified', 'featured', 'expired']) {
      if (typeof opportunity[field] !== 'boolean') errors.push(`${label}: ${field} must be boolean`);
    }

    for (const field of ['eligibility', 'documents_needed', 'tags']) {
      if (!Array.isArray(opportunity[field]) || opportunity[field].length === 0) {
        errors.push(`${label}: ${field} must be a non-empty array`);
      }
    }

    if (typeof opportunity.description !== 'string' || opportunity.description.length < 80) {
      warnings.push(`${label}: description should be at least 80 characters for search quality`);
    }
  }
}

function validateGuides(guides) {
  if (!assertArray('guides', guides, 'data/guides.json')) return;

  checkUnique(guides, 'id', 'guides');
  checkUnique(guides, 'slug', 'guides');

  for (const guide of guides) {
    const label = `guide ${guide.id || guide.slug || '(unknown)'}`;
    checkRequired(guide, guideRequiredFields, label);

    if (!isSlug(guide.slug)) errors.push(`${label}: slug must be lowercase kebab-case`);
    if (!guideCategories.has(guide.category)) errors.push(`${label}: unknown category "${guide.category}"`);

    for (const field of ['published_date', 'updated_date']) {
      if (!isIsoDate(guide[field])) errors.push(`${label}: ${field} must be YYYY-MM-DD`);
    }

    if (guide.updated_date && guide.published_date && guide.updated_date < guide.published_date) {
      errors.push(`${label}: updated_date cannot be before published_date`);
    }

    if (!Array.isArray(guide.faq) || guide.faq.length === 0) {
      errors.push(`${label}: faq must be a non-empty array`);
    }

    if (typeof guide.meta_description === 'string' && guide.meta_description.length > 160) {
      warnings.push(`${label}: meta_description is over 160 characters`);
    }
  }
}

function validateMetadata(file, requiredFields) {
  const records = readJson(file);
  if (!records || !assertArray(file, records, file)) return;

  checkUnique(records, 'id', file);
  for (const record of records) {
    const label = `${file} ${record.id || '(unknown)'}`;
    for (const field of requiredFields) {
      if (!(field in record)) errors.push(`${label}: missing required field "${field}"`);
    }
    if (!isSlug(record.id)) errors.push(`${label}: id must be lowercase kebab-case`);
  }
}

if (!fs.existsSync(DATA_DIR)) {
  errors.push('Expected lowercase data/ directory. Rename Data/ to data/ for Linux and GitHub Pages compatibility.');
}

validateOpportunities(readJson('data/opportunities.json'));
validateGuides(readJson('data/guides.json'));
validateMetadata('data/categories.json', ['id', 'label', 'icon', 'description']);
validateMetadata('data/provinces.json', ['id', 'label', 'icon', 'major_cities', 'description']);

for (const warning of warnings) console.warn(`⚠️  ${warning}`);

if (errors.length > 0) {
  console.error('\n❌ Content validation failed:');
  for (const error of errors) console.error(`   - ${error}`);
  process.exit(1);
}

console.log(`✅ Content validation passed with ${warnings.length} warning(s).`);
````

## File: .gitignore
````
# ═══════════════════════════════════════════════════════════
# OpportunitiesZA — .gitignore
# Files that should NEVER be pushed to the public GitHub repo
# ═══════════════════════════════════════════════════════════

# ── ADMIN TOOLS (local only — keep private) ──────────────────
admin-uploader.html
social-media-generator.html
seo-dashboard.html
content-planner.html
PROJECT_INDEX.html

# ── DEPLOY SCRIPT (optional — has no secrets but local-use) ──
# deploy.sh          # Uncomment this line to also exclude the deploy script

# ── ENVIRONMENT / SECRETS ────────────────────────────────────
.env
.env.local
.env.production
*.secret

# ── DOCUMENTATION PARTS (large — keep locally) ───────────────
# Uncomment to exclude docs from repo (recommended for size):
# CareerOpportunitySA_ProjectDocs*.html

# ── SYSTEM FILES ─────────────────────────────────────────────
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
Thumbs.db
desktop.ini

# ── EDITOR / IDE ─────────────────────────────────────────────
.vscode/
.idea/
*.swp
*.swo
*~

# ── NODE (if you use sitemap generator) ──────────────────────
node_modules/
npm-debug.log*
package-lock.json

# ── TEMPORARY FILES ──────────────────────────────────────────
*.tmp
*.temp
*.bak
*.backup

# ── PRIVATE NOTES ────────────────────────────────────────────
PRIVATE.md
NOTES.md
TODO_PRIVATE.md
passwords.txt
credentials.txt

# ═══════════════════════════════════════════════════════════
# FILES THAT SHOULD BE IN THE REPO (do not add these):
# ✅ index.html (or OpportunitiesZA_Website.html renamed)
# ✅ 404.html
# ✅ robots.txt
# ✅ sitemap.xml (generated)
# ✅ CNAME (custom domain)
# ✅ README.md
# ✅ sitemap-generator.js
# ✅ deploy.sh (optional)
# ✅ /data/*.json
# ✅ /assets/* (images, css, js)
# ═══════════════════════════════════════════════════════════
````

## File: 404.html
````html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>Page Not Found — OpportunitiesZA</title>
<script>
  // GitHub Pages SPA routing fix:
  // 404.html captures the path and redirects to index.html
  // which reads the stored path and navigates to the right page.
  const path = window.location.pathname;
  // Only redirect if this is a SPA route (not a file with extension)
  if (!path.match(/\.\w{2,4}$/)) {
    sessionStorage.setItem('__redirect_path', path);
    // Redirect to the root — adjust if your repo is in a subdirectory
    window.location.replace('/');
  }
</script>
<style>
  *{box-sizing:border-box;margin:0;padding:0}
  body{font-family:'DM Sans',system-ui,sans-serif;background:#f8fafc;color:#1a1a1a;
    display:flex;align-items:center;justify-content:center;min-height:100vh;padding:24px;text-align:center}
  .card{background:#fff;border:1px solid #e5e7eb;border-radius:16px;padding:48px 40px;max-width:480px;box-shadow:0 4px 24px rgba(0,0,0,.08)}
  .icon{font-size:64px;margin-bottom:20px;display:block}
  h1{font-size:72px;font-weight:900;color:#0d4a26;line-height:1;margin-bottom:8px}
  h2{font-size:22px;font-weight:700;color:#111;margin-bottom:12px}
  p{font-size:15px;color:#6b7280;line-height:1.7;margin-bottom:28px}
  a{display:inline-block;padding:13px 28px;background:#0d4a26;color:#fff;border-radius:9px;font-weight:700;font-size:15px;text-decoration:none}
  a:hover{background:#083318}
  .sub{font-size:12px;color:#9ca3af;margin-top:20px}
</style>
</head>
<body>
<div class="card">
  <span class="icon">🧭</span>
  <h1>404</h1>
  <h2>Page not found</h2>
  <p>This page doesn't exist — but thousands of verified learnerships, bursaries, and internships do. Let's get you back on track.</p>
  <a href="/">Back to Opportunities</a>
  <p class="sub">OpportunitiesZA · South Africa's Career Opportunity Platform</p>
</div>
</body>
</html>
````

## File: CareerHub SA AI Development Strategy.txt
````
uSETA Careers Platform, you can go much further than simply using smaller prompts. Based on the optimization work we've already developed for this project, the goal is to create a repository that is intentionally designed for AI-assisted development. That means Codex should never need to rediscover the architecture, coding standards, workflow, or project rules on every task.

The strategy below adapts the general workflow specifically to your repository and incorporates the context optimization framework we've been building.


---

CareerHub SA AI Development Strategy

Objective

Transform the repository into an AI-optimized project where Codex:

reads only the files required

never scans the whole repository

understands project architecture from documentation instead of source code

performs exactly one task

validates its work

commits

stops


The objective is to reduce:

context usage

prompt size

execution time

hallucinations

unintended edits


while improving consistency.


---

Phase 1 — Create an AI-Friendly Repository

This is the foundation.

Instead of relying on long prompts, move permanent knowledge into the repository.

Example structure:

docs/

    architecture/

    context/

    rules/

    workflows/

    templates/

    phases/

    modules/

    prompts/

These folders become Codex's knowledge base.


---

Step 1.1 — Architecture Documentation

Create documentation describing the project instead of forcing Codex to infer it.

Example:

docs/architecture/

    overview.md

    frontend.md

    backend.md

    crawler.md

    parser.md

    ai.md

    publishing.md

Each document should explain:

purpose

responsibilities

dependencies

public interfaces

prohibited modifications


Instead of scanning 300 files, Codex reads one document.


---

Step 1.2 — Module Documentation

Every major module should have its own context.

Example

docs/modules/

    authentication.md

    search.md

    crawler.md

    parser.md

    staging.md

    review.md

    admin.md

    seo.md

Each document includes

Purpose

Entry files

Imports

Outputs

Dependencies

Known limitations

Future tasks

This replaces repository discovery.


---

Step 1.3 — Repository Rules

Create permanent project rules.

Example

docs/rules/

    coding.md

    naming.md

    testing.md

    imports.md

    commits.md

    boundaries.md

Instead of repeating

"Do not rename files"

in every prompt,

Codex reads the rule once.


---

Phase 2 — Split Development into Independent Modules

One of the largest sources of token usage is feature overlap.

Instead of thinking about the project as one application, divide it into isolated systems.

For CareerHub SA, an example breakdown is:

Module 01

Authentication

Module 02

Homepage

Module 03

Navigation

Module 04

Search

Module 05

Opportunity Details

Module 06

Bookmarks

Module 07

SETA Directory

Module 08

Province Pages

Module 09

Crawler

Module 10

Parser

Module 11

Review Queue

Module 12

Publishing

Module 13

Admin Dashboard

Module 14

SEO

Module 15

Deployment

Each module should expose clear entry points and dependencies so Codex doesn't need to inspect unrelated areas.


---

Phase 3 — Build a Development Backlog

Instead of asking Codex to "improve the project", create a backlog of small, independent tickets.

Each ticket should implement a single capability.

For example:

Task 001

Create navigation bar

Task 002

Responsive mobile menu

Task 003

Homepage hero

Task 004

Homepage search bar

Task 005

Keyword search

Task 006

Province filter

Task 007

SETA filter

Task 008

Pagination

Task 009

Opportunity detail page

Task 010

Bookmark button

As the platform grows, expand this into tasks for the crawler, parser, AI review, publishing, admin, analytics, and deployment.


---

Phase 4 — Define Repository Boundaries

Every task must explicitly state what Codex is allowed to inspect.

For each task specify:

Allowed directories

src/features/search/

src/components/search/

src/hooks/

src/lib/search/

Forbidden directories

admin/

scripts/

docs/

.github/

crawler/

parser/

tests/

package.json

Codex should only follow imports recursively from the allowed entry points.


---

Phase 5 — Create Context Files

Rather than embedding architecture in prompts, maintain lightweight context documents.

Suggested structure:

docs/context/

    repository-summary.md

    frontend-context.md

    backend-context.md

    search-context.md

    crawler-context.md

    parser-context.md

    admin-context.md

Each file should answer:

What is this module?

Where does it begin?

Which files are entry points?

What depends on it?

What must never be changed?


Codex reads these instead of exploring the codebase.


---

Phase 6 — Standardize Task Prompts

Create reusable prompt templates under:

docs/templates/

Include templates for:

Feature implementation

Bug fix

Refactoring

Testing

Documentation

Performance optimization

Dependency updates


Each template should already contain:

role

objective

repository scope

entry files

success criteria

validation steps

stop condition


This minimizes prompt repetition and token usage.


---

Phase 7 — Enforce Single-Feature Tasks

Each task should implement exactly one feature.

For example:

Good

Province filter

Search pagination

Opportunity card

Saved opportunities

Login form


Avoid

Improve search

Refactor homepage

Enhance UI

Optimize application


Specificity reduces unnecessary code exploration.


---

Phase 8 — Use Dependency Maps

Document feature dependencies so Codex understands what is required before implementing a module.

For example:

Search

depends on

↓

Opportunity model

↓

API client

↓

Search hook

↓

Search filters

↓

Pagination

Codex only reads the documented dependencies instead of tracing them across the repository.


---

Phase 9 — Define Validation Workflows

Every task should end with the same sequence:

1. Run lint.


2. Run tests relevant to the modified files.


3. Fix failures introduced by the task.


4. Ensure no unrelated files changed.


5. Commit using the project's commit convention.


6. Stop and wait for the next task.



This creates predictable completion criteria and prevents Codex from drifting into additional work.


---

Phase 10 — Use Android as a Task Manager

When away from your development machine, your phone becomes a lightweight orchestration tool rather than a coding environment.

Typical workflow:

1. Open the Codex task interface.


2. Select the next backlog item.


3. Supply only the relevant template and repository boundaries.


4. Let Codex complete the work.


5. Review the changed files.


6. Approve or request revisions.


7. Queue the next task.



Because the repository already contains architecture, rules, context, and templates, prompts remain short while still giving Codex everything it needs.


---

Phase 11 — Evolve the Repository into an AI-Native Codebase

The optimization framework you've been building should become a permanent part of the project. Your repository should contain:

docs/
├── architecture/
├── context/
├── modules/
├── phases/
├── prompts/
├── rules/
├── templates/
└── workflows/

Each directory has a specific purpose:

architecture/ — Explains the overall system design and how components interact.

context/ — Provides concise AI-readable summaries of each subsystem to avoid source-code discovery.

modules/ — Documents responsibilities, entry points, dependencies, and interfaces for each feature area.

phases/ — Breaks development into sequential implementation stages with defined prerequisites.

prompts/ — Stores reusable master prompts and task-specific prompt patterns.

rules/ — Defines coding standards, repository boundaries, commit conventions, testing expectations, and AI operating constraints.

templates/ — Contains standardized task templates for feature work, bug fixes, refactoring, testing, documentation, and reviews.

workflows/ — Documents end-to-end development processes, validation pipelines, release procedures, and AI-assisted development workflows.


Why this strategy fits your project

This aligns directly with the optimization framework we've already been developing for the CareerHub SA repository. Instead of relying on increasingly complex prompts as the platform grows to include authentication, search, SETA directories, province pages, crawler pipelines, AI parsing, review queues, publishing workflows, an admin dashboard, SEO, and deployment, the repository itself becomes the primary source of context.

The result is a scalable workflow where Codex receives a single, well-scoped ticket, reads only the required documentation and code, completes one implementation, validates it, commits the change, and stops. As the repository expands, token usage remains predictable because architecture, rules, and workflow knowledge are stored once in the repository rather than being repeated in every prompt. This is the long-term strategy that will allow your project to scale efficiently while maintaining high-quality AI-assisted development.


part 2 recently add 
############### ############### ############################################################
Repository Workflow Optimization Framework
Codex Context / Token Usage Reduction Framework
Based on the conversations in this project, there are two major themes that were developed in depth:

1. Repository Workflow Optimization Framework


2. Codex Context / Token Usage Reduction Framework



These discussions evolved into a methodology for building large repositories with AI while minimizing unnecessary context loading, reducing prompt sizes, increasing task completion rates, and making future AI sessions significantly cheaper and more accurate.


---

PART I — Repository Workflow Optimization Framework

The repository workflow was designed around a modular AI-first development process.

Instead of treating the repository as one large project, it is divided into independent systems.

The goal is:

minimize AI context

isolate failures

improve maintainability

allow parallel development

reduce hallucinations

improve debugging

improve testing



---

1 Repository as Independent Systems

Rather than:

Repository
    ↓
Everything connected

the repository becomes

Repository

Frontend
Backend
Data Sources
Scrapers
API
Validation
Scheduler
Parser
Documentation
Tests
Deployment

Each system owns its own:

documentation

architecture

tests

configuration

AI context


Meaning Codex only loads the section being modified.


---

2 Layered Architecture

Development is separated into layers.

Example

UI

↓

Business Logic

↓

Services

↓

Data Layer

↓

Storage

↓

Configuration

Each layer communicates through interfaces rather than directly.

Benefits

fewer dependencies

easier debugging

smaller AI context

isolated testing



---

3 Feature Isolation

Every feature is treated as its own module.

Instead of

Everything imports everything

Use

Feature A

docs
tests
logic
config

Feature B

docs
tests
logic
config

AI only needs Feature B to modify Feature B.


---

4 Task-Based Development

Large prompts become hundreds of small tasks.

Example

Instead of

Build scraping system

Split into

Task 1

Create source loader

Task 2

Validate sources

Task 3

Create parser interface

Task 4

Implement PDF parser

Task 5

Unit tests

Each task loads dramatically fewer files.


---

5 Strict Repository Boundaries

Every task clearly defines

Allowed folders

Forbidden folders

Files to modify

Files to create

Expected outputs

Success criteria

Rollback strategy

This prevents Codex from exploring the entire repository.


---

6 Documentation-Driven Development

Documentation becomes the primary source of truth.

Examples

docs/

architecture

modules

rules

context

workflow

templates

contracts

interfaces

Instead of loading 300 source files, AI loads a few documentation files.


---

7 Phase-Based Development

Development progresses in phases.

Example

Phase 0

Repository structure

Phase 1

Infrastructure

Phase 2

Core engine

Phase 3

Scrapers

Phase 4

API

Phase 5

UI

Phase 6

Testing

Phase 7

Deployment

Each phase has independent documentation.


---

PART II — Codex Token Usage Reduction Framework

This became one of the major focuses of the project.

Goal:

Reduce context size while increasing output quality.


---

Core Principle

Never make Codex understand the whole repository.

Instead

Make Codex understand one task.


---

1 Context Isolation

Every task receives only

Objective

Files

Dependencies

Expected outputs

Success criteria

Nothing else.

Instead of

Understand entire repository.

Provide

Modify:

scripts/parser.js

Only.

Huge token reduction.


---

2 Context Hierarchy

The framework defines multiple context levels.

Repository Context

↓

Module Context

↓

Feature Context

↓

Task Context

↓

File Context

↓

Function Context

Codex should only receive the lowest necessary level.


---

3 Context Files

Dedicated documentation files were proposed.

Examples

docs/context/

repository.md

frontend.md

backend.md

parser.md

api.md

scraper.md

scheduler.md

validation.md

Codex reads only one context file.


---

4 AI Rules

Create

docs/rules/

Containing

Naming conventions

Folder rules

Coding standards

Architecture rules

Import rules

Testing rules

Documentation rules

Instead of explaining rules every prompt.


---

5 Prompt Templates

Instead of rewriting prompts.

Store templates.

Example

docs/templates/

bug-fix.md

feature.md

refactor.md

documentation.md

testing.md

review.md

optimization.md


---

6 Scope Limitation

Every task contains

Scope

Out of scope

Forbidden actions

Example

ONLY modify

scripts/parser.js

Do NOT modify

data

UI

tests

package.json

Codex explores dramatically fewer files.


---

7 Explicit Constraints

Tasks define

May create

May edit

Must not edit

Allowed dependencies

Forbidden dependencies

Expected filenames

Expected functions

Maximum files modified

This prevents unnecessary exploration.


---

8 Small Independent Tasks

Large

Implement scraper

becomes

Task A

Create interface

Task B

Validate URLs

Task C

Normalize output

Task D

Error handling

Task E

Tests

Task F

Documentation

Each prompt becomes tiny.


---

9 Task Chaining

Rather than

Huge prompt

Use

Task 1

↓

Task 2

↓

Task 3

↓

Task 4

Each task references outputs from previous tasks instead of repeating full context.


---

10 Repository Memory

Persistent documentation replaces repeated explanations.

Examples

architecture.md

coding-rules.md

interfaces.md

contracts.md

Codex loads these instead of rediscovering the repository.


---

11 Interface Contracts

Modules communicate through documented interfaces.

Example

Input

↓

Validation

↓

Parser

↓

Normalizer

↓

Storage

Each interface is documented.

Codex doesn't inspect every module.


---

12 Stable APIs

Document

Inputs

Outputs

Types

Errors

Examples

No source-code exploration required.


---

13 AI Context Compression

Instead of

100 files

Summarize into

One architecture file

One interface file

One workflow file

One module file

This dramatically reduces prompt size.


---

14 Documentation Before Code

Before implementation create

Architecture

Contracts

Interfaces

Workflow

Dependencies

Then implementation.

AI reads documentation instead of discovering code.


---

15 Standard Task Structure

Every Codex prompt follows a fixed structure.

1. Objective


2. Scope


3. Files allowed


4. Files forbidden


5. Expected output


6. Constraints


7. Acceptance criteria



This standardization reduces ambiguity and unnecessary context.


---

16 Dependency Mapping

Document module dependencies explicitly.

Instead of allowing AI to infer relationships, provide a dependency graph showing which modules interact and which are isolated. This reduces exploratory code reading.


---

17 Incremental Repository Indexing

Maintain lightweight index files that summarize repository structure, major modules, and entry points. AI consults these indexes first before any code, avoiding repeated directory traversal.


---

18 Architecture Decision Records (ADRs)

Capture important architectural decisions in dedicated documents. Future tasks reference ADRs instead of rediscovering why design choices were made, preventing repeated analysis.


---

19 Success Criteria-Driven Prompts

Every task specifies measurable completion criteria such as:

Required files created or modified.

Tests passing.

Documentation updated.

No unrelated files changed.


This limits over-implementation.


---

20 Failure and Rollback Guidance

Include expected failure handling and rollback instructions within task prompts so Codex can avoid cascading changes when a step cannot be completed.


---

21 Documentation Directory Structure

A dedicated documentation hierarchy was proposed to support context compression:

docs/
├── architecture/
├── context/
├── rules/
├── templates/
├── workflow/
├── interfaces/
├── contracts/
├── modules/
├── phases/
├── testing/
├── deployment/
└── decisions/

Each directory serves a distinct purpose, allowing AI to load only the documentation relevant to the current task.


---

22 Prompt Design Principles

The discussions consistently emphasized that effective prompts should:

Be single-objective.

Have explicit scope boundaries.

Define allowed and forbidden files.

Specify expected outputs and acceptance criteria.

Avoid repeating repository-wide context.

Reference existing documentation instead of embedding large explanations.

Be reusable through standardized templates.


---

23 Expected Benefits

Implementing the combined Repository Workflow Optimization Framework and Codex Token Usage Reduction Framework is expected to provide:

Significant reductions in prompt and context token consumption.

Lower AI operating costs.

Faster task completion.

Higher completion accuracy.

Reduced hallucinations.

Fewer unintended repository modifications.

Improved maintainability and onboarding.

Better modularity and separation of concerns.

Easier testing and debugging.

Greater consistency across AI-assisted development sessions.

Reusable documentation and prompt assets that compound efficiency over time.


Together, these discussions define an AI-first repository engineering methodology: structure the repository around modular components, capture architecture and rules in persistent documentation, constrain every AI task to the smallest practical scope, and standardize prompts so Codex operates with minimal context while producing predictable, high-quality results.
````

## File: CNAME
````
setacareersportal.abrdns.com
setacareersportal.abrdns.com
````

## File: deploy.sh
````bash
#!/bin/bash
# ═══════════════════════════════════════════════════════════════
# OpportunitiesZA — Termux Deploy & Maintenance Script
# Save as: deploy.sh
# Make executable: chmod +x deploy.sh
# Run: ./deploy.sh
# ═══════════════════════════════════════════════════════════════

# ── CONFIG ────────────────────────────────────────────────────
PROJECT_DIR="/storage/emulated/0/Download/opportunitiesza"
REPO_URL="https://github.com/Ituthema/seta-careers-platform.git"
BRANCH="main"

# Colors
GREEN="\033[0;32m"
AMBER="\033[0;33m"
RED="\033[0;31m"
CYAN="\033[0;36m"
WHITE="\033[1;37m"
RESET="\033[0m"

# ── HEADER ────────────────────────────────────────────────────
clear
echo ""
echo -e "${GREEN}╔══════════════════════════════════════════════════════╗${RESET}"
echo -e "${GREEN}║${WHITE}  🧭 OpportunitiesZA — Deploy & Maintenance Script  ${GREEN}  ║${RESET}"
echo -e "${GREEN}║${AMBER}     South Africa's Career Opportunity Platform       ${GREEN}  ║${RESET}"
echo -e "${GREEN}╚══════════════════════════════════════════════════════╝${RESET}"
echo ""

# ── CHECK PROJECT DIR ─────────────────────────────────────────
if [ ! -d "$PROJECT_DIR" ]; then
  echo -e "${RED}❌ Project directory not found: $PROJECT_DIR${RESET}"
  echo -e "${AMBER}   Edit PROJECT_DIR at the top of this script.${RESET}"
  exit 1
fi

cd "$PROJECT_DIR" || exit 1
echo -e "${CYAN}📁 Working directory: $PROJECT_DIR${RESET}"
echo ""

# ── MAIN MENU ─────────────────────────────────────────────────
echo -e "${WHITE}What would you like to do?${RESET}"
echo ""
echo "  1) 🚀 Quick publish (add all, commit, push)"
echo "  2) 📝 Add new opportunity (guided)"
echo "  3) ✅ Validate JSON files"
echo "  4) 🗑️  Mark opportunity as expired"
echo "  5) 🗺️  Regenerate sitemap.xml"
echo "  6) 📊 Show project stats"
echo "  7) 🔄 Pull latest from GitHub"
echo "  8) 🏗️  Initial setup (first time)"
echo "  9) ❌ Exit"
echo ""
read -r -p "Enter choice [1-9]: " choice

echo ""

case $choice in

# ── 1. QUICK PUBLISH ─────────────────────────────────────────
1)
  echo -e "${CYAN}🚀 Quick Publish${RESET}"
  echo ""

  # Show what will be committed
  echo -e "${AMBER}Changes to commit:${RESET}"
  git status --short
  echo ""

  if [ -z "$(git status --porcelain)" ]; then
    echo -e "${AMBER}ℹ️  Nothing to commit — working tree is clean.${RESET}"
    exit 0
  fi

  # Validate JSON before pushing
  echo -e "${CYAN}Validating JSON files...${RESET}"
  for f in data/*.json; do
    if python3 -m json.tool "$f" > /dev/null 2>&1; then
      echo -e "  ${GREEN}✅ $f — valid${RESET}"
    else
      echo -e "  ${RED}❌ $f — INVALID JSON! Fix this before pushing.${RESET}"
      python3 -m json.tool "$f"
      exit 1
    fi
  done
  echo ""

  # Commit message
  read -r -p "Commit message (or press Enter for auto): " msg
  if [ -z "$msg" ]; then
    msg="Content update $(date '+%Y-%m-%d %H:%M')"
  fi

  git add .
  git commit -m "$msg"

  echo ""
  echo -e "${CYAN}Pushing to GitHub...${RESET}"
  if git push origin "$BRANCH"; then
    echo ""
    echo -e "${GREEN}✅ Pushed successfully!${RESET}"
    echo -e "${CYAN}   Site updates in 1–5 minutes.${RESET}"
    echo -e "${CYAN}   Check: https://github.com/Ituthema/seta-careers-platform/actions${RESET}"
  else
    echo -e "${RED}❌ Push failed. Check your internet connection or GitHub credentials.${RESET}"
    exit 1
  fi
  ;;

# ── 2. ADD NEW OPPORTUNITY ────────────────────────────────────
2)
  echo -e "${CYAN}📝 Add New Opportunity${RESET}"
  echo ""

  DATA_FILE="data/opportunities.json"

  if [ ! -f "$DATA_FILE" ]; then
    echo -e "${RED}❌ $DATA_FILE not found.${RESET}"
    exit 1
  fi

  # Gather details
  read -r -p "Title:              " title
  echo "Category (learnership/internship/bursary/apprenticeship):"
  read -r -p "Category:           " cat
  echo "Province (Nationwide/Gauteng/Western Cape/KwaZulu-Natal/Eastern Cape/Limpopo/Mpumalanga/North West/Free State/Northern Cape):"
  read -r -p "Province:           " prov
  read -r -p "Qualification (Matric/Diploma/Degree): " qual
  read -r -p "Stipend:            " stipend
  read -r -p "Closing date (YYYY-MM-DD): " close
  read -r -p "Application URL:    " url
  read -r -p "Source name:        " srcname
  read -r -p "Source URL:         " srcurl
  read -r -p "Description (1-2 sentences): " desc
  read -r -p "Tags (comma-separated, e.g. matric,gauteng): " tags
  read -r -p "Verified? (y/n):    " verif

  # Build ID from timestamp
  id=$(date +%s)
  today=$(date '+%Y-%m-%d')

  # Build slug from title
  slug=$(echo "$title" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9 ]//g' | sed 's/ \+/-/g' | sed 's/-\+/-/g' | sed 's/^-//;s/-$//')

  verified="true"
  [ "$verif" = "n" ] && verified="false"

  # Build JSON record
  record=$(cat <<EOF
{
  "id": "$id",
  "slug": "$slug",
  "title": "$title",
  "category": "$cat",
  "province": "$prov",
  "qualification_level": "$qual",
  "stipend": "$stipend",
  "closing_date": "$close",
  "application_url": "$url",
  "source_name": "$srcname",
  "source_url": "$srcurl",
  "verified": $verified,
  "featured": false,
  "expired": false,
  "description": "$desc",
  "eligibility": ["South African citizen", "Valid ID", "$qual certificate"],
  "documents_needed": ["Certified ID", "Certified $qual certificate", "CV", "Motivational letter"],
  "tags": [$(echo "$tags" | sed 's/,/","/g' | sed 's/^/"/' | sed 's/$/"/')],
  "posted_date": "$today",
  "updated_date": "$today"
}
EOF
)

  echo ""
  echo -e "${AMBER}Record to be added:${RESET}"
  echo "$record"
  echo ""
  read -r -p "Add this to $DATA_FILE? (y/n): " confirm

  if [ "$confirm" = "y" ]; then
    # Insert after the opening bracket of the JSON array
    tmp=$(mktemp)
    python3 -c "
import json, sys

with open('$DATA_FILE', 'r') as f:
    data = json.load(f)

new_record = json.loads('''$record''')
data.insert(0, new_record)

with open('$DATA_FILE', 'w') as f:
    json.dump(data, f, indent=2, ensure_ascii=False)
print('✅ Record added successfully.')
"
    if [ $? -eq 0 ]; then
      echo ""
      echo -e "${GREEN}✅ Opportunity added to $DATA_FILE${RESET}"
      read -r -p "Push to GitHub now? (y/n): " dopush
      if [ "$dopush" = "y" ]; then
        git add "$DATA_FILE"
        git commit -m "Add opportunity: $title (closes $close)"
        git push origin "$BRANCH"
        echo -e "${GREEN}✅ Pushed!${RESET}"
      fi
    fi
  else
    echo -e "${AMBER}Cancelled — nothing was changed.${RESET}"
  fi
  ;;

# ── 3. VALIDATE JSON ─────────────────────────────────────────
3)
  echo -e "${CYAN}✅ Validating all JSON data files...${RESET}"
  echo ""
  errors=0
  for f in data/*.json; do
    if [ -f "$f" ]; then
      if python3 -m json.tool "$f" > /dev/null 2>&1; then
        count=$(python3 -c "import json; d=json.load(open('$f')); print(len(d) if isinstance(d,list) else 'object')")
        echo -e "  ${GREEN}✅ $f — valid ($count records)${RESET}"
      else
        echo -e "  ${RED}❌ $f — INVALID JSON${RESET}"
        echo -e "  ${AMBER}Error details:${RESET}"
        python3 -m json.tool "$f" 2>&1 | head -5
        errors=$((errors + 1))
      fi
    fi
  done
  echo ""
  if [ $errors -eq 0 ]; then
    echo -e "${GREEN}✅ All JSON files are valid!${RESET}"
    if command -v node &>/dev/null && [ -f validate-content.js ]; then
      echo ""
      echo -e "${CYAN}Checking content/sitemap slug agreement...${RESET}"
      if node validate-content.js; then
        echo -e "${GREEN}✅ Content validation passed!${RESET}"
      else
        echo -e "${RED}❌ Content validation failed. Run node sitemap-generator.js and re-check.${RESET}"
        exit 1
      fi
    fi
  else
    echo -e "${RED}❌ $errors file(s) have errors. Fix them before pushing.${RESET}"
  fi
  ;;

# ── 4. MARK EXPIRED ──────────────────────────────────────────
4)
  echo -e "${CYAN}🗑️  Mark Opportunity as Expired${RESET}"
  echo ""
  echo -e "${AMBER}Current live opportunities:${RESET}"
  python3 -c "
import json
from datetime import date
with open('data/opportunities.json') as f:
    opps = json.load(f)
live = [o for o in opps if not o.get('expired', False)]
for i, o in enumerate(live):
    closing = o.get('closing_date','')
    try:
        days = (date.fromisoformat(closing) - date.today()).days
        status = f'{days}d left' if days >= 0 else 'OVERDUE'
    except:
        status = 'unknown'
    print(f'  [{i+1}] {o[\"title\"]} ({status})')
"
  echo ""
  read -r -p "Enter number to mark as expired (or 0 to cancel): " num

  if [ "$num" -gt 0 ] 2>/dev/null; then
    python3 -c "
import json
with open('data/opportunities.json') as f:
    opps = json.load(f)
live = [o for o in opps if not o.get('expired', False)]
idx = $num - 1
if 0 <= idx < len(live):
    slug = live[idx]['slug']
    title = live[idx]['title']
    for o in opps:
        if o.get('slug') == slug:
            o['expired'] = True
            print(f'Marked as expired: {title}')
    with open('data/opportunities.json', 'w') as f:
        json.dump(opps, f, indent=2, ensure_ascii=False)
else:
    print('Invalid selection.')
"
    read -r -p "Push change to GitHub? (y/n): " dopush
    if [ "$dopush" = "y" ]; then
      git add data/opportunities.json
      git commit -m "Archive expired opportunity"
      git push origin "$BRANCH"
      echo -e "${GREEN}✅ Pushed!${RESET}"
    fi
  else
    echo -e "${AMBER}Cancelled.${RESET}"
  fi
  ;;

# ── 5. REGENERATE SITEMAP ─────────────────────────────────────
5)
  echo -e "${CYAN}🗺️  Regenerating sitemap.xml...${RESET}"
  if command -v node &>/dev/null; then
    node sitemap-generator.js
    if [ -f validate-content.js ]; then
      node validate-content.js || exit 1
    fi
    read -r -p "Push sitemap to GitHub? (y/n): " dopush
    if [ "$dopush" = "y" ]; then
      git add sitemap.xml
      git commit -m "Update sitemap $(date '+%Y-%m-%d')"
      git push origin "$BRANCH"
      echo -e "${GREEN}✅ Sitemap pushed!${RESET}"
    fi
  else
    echo -e "${RED}❌ Node.js not installed. Run: pkg install nodejs${RESET}"
  fi
  ;;

# ── 6. PROJECT STATS ──────────────────────────────────────────
6)
  echo -e "${CYAN}📊 Project Statistics${RESET}"
  echo ""
  python3 -c "
import json
from datetime import date

try:
    with open('data/opportunities.json') as f:
        opps = json.load(f)
    live = [o for o in opps if not o.get('expired', False)]
    expired = [o for o in opps if o.get('expired', False)]
    closing_soon = [o for o in live if o.get('closing_date') and (date.fromisoformat(o['closing_date']) - date.today()).days <= 30]
    by_cat = {}
    for o in live:
        c = o.get('category', 'unknown')
        by_cat[c] = by_cat.get(c, 0) + 1
    print(f'  Total opportunities:  {len(opps)}')
    print(f'  Live (open):          {len(live)}')
    print(f'  Expired:              {len(expired)}')
    print(f'  Closing this month:   {len(closing_soon)}')
    print(f'  Verified:             {sum(1 for o in live if o.get(\"verified\"))}')
    print()
    print('  By category:')
    for cat, count in sorted(by_cat.items()):
        print(f'    {cat}: {count}')
except Exception as e:
    print(f'Error reading opportunities.json: {e}')

print()
try:
    with open('data/guides.json') as f:
        guides = json.load(f)
    print(f'  Guide articles:       {len(guides)}')
except:
    print('  Guide articles:       (guides.json not found)')
"
  echo ""
  echo -e "  Git status:"
  git log --oneline -5 | sed 's/^/    /'
  ;;

# ── 7. PULL LATEST ────────────────────────────────────────────
7)
  echo -e "${CYAN}🔄 Pulling latest changes from GitHub...${RESET}"
  git fetch origin
  git pull origin "$BRANCH"
  echo -e "${GREEN}✅ Up to date with GitHub.${RESET}"
  ;;

# ── 8. INITIAL SETUP ─────────────────────────────────────────
8)
  echo -e "${CYAN}🏗️  Initial Setup${RESET}"
  echo ""
  echo "This will initialize the git repository and make the first push."
  echo ""

  # Check if git already initialized
  if [ -d ".git" ]; then
    echo -e "${AMBER}⚠️  Git already initialized in this directory.${RESET}"
    read -r -p "Re-initialize? This will reset git history. (y/n): " reinit
    if [ "$reinit" != "y" ]; then
      echo "Cancelled."
      exit 0
    fi
    rm -rf .git
  fi

  # Fix Termux safe directory
  git config --global --add safe.directory "$PROJECT_DIR"

  git init
  git add .
  git commit -m "Initial deployment — OpportunitiesZA $(date '+%Y-%m-%d')"
  git branch -M main

  echo ""
  echo -e "${AMBER}Enter your GitHub repository URL:${RESET}"
  read -r -p "URL: " repo_url
  git remote add origin "$repo_url"

  echo ""
  echo -e "${CYAN}Pushing to GitHub...${RESET}"
  echo -e "${AMBER}You'll be prompted for your GitHub username and Personal Access Token.${RESET}"
  git push -u origin main

  echo ""
  echo -e "${GREEN}✅ Initial push complete!${RESET}"
  echo -e "${CYAN}   Enable GitHub Pages: Repository → Settings → Pages → main branch${RESET}"
  ;;

# ── 9. EXIT ──────────────────────────────────────────────────
9)
  echo -e "${AMBER}Goodbye! 🧭${RESET}"
  exit 0
  ;;

*)
  echo -e "${RED}❌ Invalid choice. Run the script again.${RESET}"
  exit 1
  ;;
esac

echo ""
echo -e "${GREEN}═══════════════════════════════════════════${RESET}"
echo -e "${GREEN}  Done — OpportunitiesZA Deploy Script      ${RESET}"
echo -e "${GREEN}═══════════════════════════════════════════${RESET}"
echo ""
````

## File: index.html
````html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>OpportunitiesZA | Learnerships, Bursaries & Internships South Africa 2026</title>
<meta name="description" content="Find verified learnerships, bursaries, internships, and apprenticeships across South Africa. Free eligibility checker, deadline calendar, scam awareness tools. Updated daily.">
<meta property="og:title" content="OpportunitiesZA | SA Career Opportunities 2026">
<meta property="og:description" content="South Africa's most complete platform for learnerships, bursaries, and internships. Verified. Free. 100+ listings.">
<meta property="og:type" content="website">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,700;0,9..144,900;1,9..144,700&family=DM+Sans:ital,wght@0,300;0,400;0,500;0,600;1,400&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
<style>
/* ═══ RESET & TOKENS ═════════════════════════════════════════ */
*{box-sizing:border-box;margin:0;padding:0}
:root{
  --g:#0d4a26;--gd:#083318;--glt:#dcfce7;--gmid:#166534;
  --a:#f59e0b;--alt:#fef3c7;--ald:#d97706;
  --bl:#1e40af;--blt:#dbeafe;
  --pu:#6b21a8;--plt:#f3e8ff;
  --rd:#dc2626;--rlt:#fee2e2;
  --bd:#e5e7eb;--mu:#6b7280;--sur:#f8fafc;--wh:#fff;
  --tx:#1a1a1a;--t2:#374151;--t3:#6b7280;
  --sh:0 1px 4px rgba(0,0,0,.06);
  --shm:0 4px 18px rgba(0,0,0,.09);
  --shl:0 8px 32px rgba(0,0,0,.13);
  --r:10px;--tr:all .18s ease;
  --mw:1220px;--cw:800px;
}
html{scroll-behavior:smooth}
body{font-family:'DM Sans',sans-serif;font-size:15px;color:var(--tx);background:#fafaf9;line-height:1.65}
a{color:var(--g)}
button,input,select,textarea{font-family:'DM Sans',sans-serif}
img{max-width:100%;height:auto}
h1,h2,h3{font-family:'Fraunces',serif}

/* ═══ LAYOUT ═════════════════════════════════════════════════ */
.wrap{max-width:var(--mw);margin:0 auto;padding:0 24px}
.cwrap{max-width:var(--cw);margin:0 auto;padding:0 24px}
.page{animation:fadeUp .22s ease}
@keyframes fadeUp{from{opacity:0;transform:translateY(8px)}to{opacity:1;transform:none}}

/* ═══ BUTTONS ════════════════════════════════════════════════ */
.btn{display:inline-flex;align-items:center;justify-content:center;gap:6px;padding:10px 20px;border-radius:8px;font-weight:600;font-size:14px;border:none;cursor:pointer;transition:var(--tr);line-height:1.3;text-decoration:none}
.btn-primary{background:var(--g);color:#fff}.btn-primary:hover{background:var(--gd);transform:translateY(-1px);box-shadow:0 4px 14px rgba(13,74,38,.25)}
.btn-amber{background:var(--a);color:#fff}.btn-amber:hover{background:var(--ald);transform:translateY(-1px)}
.btn-outline{background:transparent;color:var(--g);border:1.5px solid var(--g)}.btn-outline:hover{background:var(--glt)}
.btn-ghost{background:transparent;color:var(--t2);border:1px solid var(--bd)}.btn-ghost:hover{background:var(--sur);border-color:var(--g);color:var(--g)}
.btn-red{background:var(--rd);color:#fff}.btn-red:hover{background:#b91c1c}
.btn-sm{padding:7px 14px;font-size:13px}
.btn-lg{padding:15px 32px;font-size:16px}
.btn-full{width:100%}

/* ═══ BADGES ═════════════════════════════════════════════════ */
.badge{display:inline-block;padding:3px 11px;border-radius:20px;font-size:11px;font-weight:700;text-transform:uppercase;letter-spacing:.6px}
.badge-learnership{background:var(--glt);color:var(--gmid)}
.badge-internship{background:var(--blt);color:var(--bl)}
.badge-bursary{background:var(--alt);color:#92400e}
.badge-apprenticeship{background:var(--plt);color:var(--pu)}
.badge-tvet{background:#ecfdf5;color:#065f46}
.pill{font-family:'DM Mono',monospace;font-size:10.5px;font-weight:600;padding:3px 10px;border-radius:20px;display:inline-block}
.pill-red{background:var(--rlt);color:var(--rd)}
.pill-amber{background:var(--alt);color:#92400e}
.pill-green{background:var(--glt);color:var(--gmid)}
.pill-gray{background:#f3f4f6;color:var(--mu)}

/* ═══ NAV ════════════════════════════════════════════════════ */
#nav{position:sticky;top:0;z-index:300;background:rgba(255,255,255,.97);border-bottom:1px solid var(--bd);backdrop-filter:blur(12px);transition:box-shadow .2s}
#nav.scrolled{box-shadow:0 2px 20px rgba(0,0,0,.08)}
.nav-in{max-width:var(--mw);margin:0 auto;padding:0 24px;height:64px;display:flex;align-items:center;gap:28px}
.logo{display:flex;align-items:center;gap:10px;background:none;border:none;cursor:pointer}
.logo-icon{width:34px;height:34px;background:var(--g);border-radius:8px;display:flex;align-items:center;justify-content:center;font-size:18px;flex-shrink:0}
.logo-text{font-family:'Fraunces',serif;font-size:19px;font-weight:900;color:var(--g);letter-spacing:-.5px}
.logo-text em{color:var(--a);font-style:normal}
.nav-links{display:flex;gap:2px;margin-left:auto;align-items:center}
.nl{background:none;border:none;padding:7px 11px;border-radius:6px;font-size:13.5px;font-weight:500;color:var(--mu);cursor:pointer;transition:var(--tr)}
.nl:hover,.nl.act{background:var(--glt);color:var(--g)}
.nl-cta{margin-left:8px;padding:8px 18px;background:var(--g);color:#fff!important;border-radius:7px;font-size:13.5px;font-weight:600}
.nl-cta:hover{background:var(--gd)}
.hbg{display:none;background:none;border:none;font-size:22px;color:var(--tx);margin-left:auto;cursor:pointer}
.drawer{display:none;flex-direction:column;gap:2px;padding:12px 24px 20px;border-top:1px solid var(--bd)}
.drawer.open{display:flex}
.dl{background:none;border:none;padding:10px 12px;border-radius:6px;font-size:14px;font-weight:500;color:var(--t2);text-align:left;cursor:pointer}
.dl:hover{background:var(--sur)}

/* ═══ HERO ═══════════════════════════════════════════════════ */
.hero{background:linear-gradient(135deg,#0a3018 0%,var(--g) 55%,#116232 100%);padding:76px 24px 84px;position:relative;overflow:hidden}
.hero::before{content:'';position:absolute;top:-100px;right:-100px;width:420px;height:420px;border-radius:50%;background:rgba(245,158,11,.07);pointer-events:none}
.hero::after{content:'';position:absolute;bottom:-80px;left:-60px;width:300px;height:300px;border-radius:50%;background:rgba(255,255,255,.04);pointer-events:none}
.hero-in{max-width:800px;margin:0 auto;text-align:center;position:relative;z-index:1}
.hero-badge{display:inline-flex;align-items:center;gap:8px;background:rgba(245,158,11,.15);border:1px solid rgba(245,158,11,.3);border-radius:20px;padding:5px 16px;margin-bottom:26px}
.hero-badge span{font-family:'DM Mono',monospace;font-size:11px;color:#fbbf24;letter-spacing:1.5px;text-transform:uppercase}
.hero h1{font-size:clamp(34px,6.5vw,62px);font-weight:900;color:#fff;line-height:1.04;letter-spacing:-2px;margin-bottom:22px}
.hero h1 em{color:var(--a);font-style:italic}
.hero-sub{font-size:17px;color:#86efac;max-width:530px;margin:0 auto 38px;line-height:1.7;font-weight:300}
.search-bar{display:flex;max-width:660px;margin:0 auto 28px;background:#fff;border-radius:12px;overflow:hidden;box-shadow:var(--shl)}
.search-bar input{flex:1;padding:16px 20px;border:none;outline:none;font-size:15px;color:var(--tx)}
.search-bar select{padding:0 14px;border:none;border-left:1px solid var(--bd);outline:none;font-size:13px;color:var(--mu);background:transparent;cursor:pointer}
.search-bar button{padding:16px 28px;background:var(--a);color:#fff;border:none;font-weight:700;font-size:14px;cursor:pointer;transition:background .15s;white-space:nowrap}
.search-bar button:hover{background:var(--ald)}
.chips{display:flex;gap:10px;justify-content:center;flex-wrap:wrap}
.chip{padding:7px 16px;background:rgba(255,255,255,.12);color:#dcfce7;border:1px solid rgba(255,255,255,.2);border-radius:20px;font-size:13px;cursor:pointer;font-weight:500;transition:var(--tr)}
.chip:hover{background:rgba(255,255,255,.22)}

/* ═══ STATS BAR ══════════════════════════════════════════════ */
.stats-bar{background:#fff;border-bottom:1px solid var(--bd);padding:16px 24px}
.stats-in{max-width:var(--mw);margin:0 auto;display:flex;gap:36px;justify-content:center;flex-wrap:wrap}
.stat{display:flex;align-items:center;gap:10px}
.stat-num{font-family:'Fraunces',serif;font-size:26px;font-weight:900;color:var(--g);line-height:1}
.stat-label{font-size:12px;color:var(--mu);line-height:1.4;max-width:80px}

/* ═══ SECTION ════════════════════════════════════════════════ */
.sec-hdr{display:flex;align-items:baseline;justify-content:space-between;margin-bottom:28px}
.sec-title{font-size:clamp(22px,3.5vw,34px);font-weight:900;color:var(--g);letter-spacing:-1px}
.sec-lnk{background:none;border:none;color:var(--g);font-weight:600;font-size:14px;cursor:pointer}
.sec-lnk:hover{text-decoration:underline}

/* ═══ CATEGORY TILES ═════════════════════════════════════════ */
.cat-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:16px}
.cat-tile{background:#fff;border:1px solid var(--bd);border-radius:var(--r);padding:22px;text-align:left;cursor:pointer;transition:var(--tr);display:flex;flex-direction:column;gap:10px}
.cat-tile:hover{border-color:var(--g);transform:translateY(-3px);box-shadow:var(--shm)}
.ct-icon{font-size:30px}
.ct-name{font-family:'Fraunces',serif;font-size:18px;font-weight:700;color:var(--tx)}
.ct-desc{font-size:12.5px;color:var(--mu);line-height:1.5}
.ct-count{margin-top:auto;font-family:'DM Mono',monospace;font-size:11px;padding:3px 10px;border-radius:10px;display:inline-block;font-weight:600}

/* ═══ CLOSING STRIP ══════════════════════════════════════════ */
.strip{background:#fffbeb;border-top:2px solid var(--a);border-bottom:1px solid #fde68a;padding:18px 24px}
.strip-in{max-width:var(--mw);margin:0 auto}
.strip-hdr{display:flex;align-items:center;gap:10px;margin-bottom:14px}
.strip-hdr span:last-child{font-family:'Fraunces',serif;font-weight:700;font-size:14px;color:#92400e;text-transform:uppercase;letter-spacing:.8px}
.strip-scroll{display:flex;gap:12px;overflow-x:auto;padding-bottom:4px;scrollbar-width:none}
.strip-scroll::-webkit-scrollbar{display:none}
.strip-pill{flex-shrink:0;background:#fff;border:1px solid #fde68a;border-radius:var(--r);padding:11px 16px;cursor:pointer;max-width:215px;transition:var(--tr);text-align:left}
.strip-pill:hover{border-color:var(--a)}
.sp-title{font-size:12px;font-weight:600;color:var(--tx);display:block;margin-bottom:4px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.sp-meta{font-size:10.5px;color:var(--mu);margin-bottom:4px}

/* ═══ OPP CARD ═══════════════════════════════════════════════ */
.opp-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(300px,1fr));gap:18px}
.opp-card{background:#fff;border:1px solid var(--bd);border-radius:var(--r);padding:20px;display:flex;flex-direction:column;gap:10px;cursor:pointer;transition:var(--tr);box-shadow:var(--sh);position:relative}
.opp-card:hover{border-color:var(--g);box-shadow:0 8px 28px rgba(13,74,38,.12);transform:translateY(-3px)}
.opp-card.featured::before{content:'⭐ Featured';position:absolute;top:12px;right:12px;font-size:10px;background:var(--alt);color:#92400e;padding:2px 8px;border-radius:10px;font-weight:700;font-family:'DM Mono',monospace;letter-spacing:.5px}
.oc-top{display:flex;justify-content:space-between;align-items:flex-start}
.oc-verified{font-size:11px;color:var(--gmid);font-weight:600}
.oc-title{font-family:'Fraunces',serif;font-size:16.5px;font-weight:700;color:var(--tx);line-height:1.3}
.oc-meta{display:flex;gap:12px;flex-wrap:wrap}
.oc-meta span{font-size:12px;color:var(--mu)}
.oc-footer{display:flex;justify-content:space-between;align-items:center;margin-top:auto;padding-top:10px;border-top:1px solid #f3f4f6}
.oc-link{font-size:13px;color:var(--g);font-weight:600}
.bk-btn{background:none;border:none;font-size:18px;cursor:pointer;padding:4px;line-height:1;opacity:.6;transition:var(--tr)}
.bk-btn:hover{opacity:1;transform:scale(1.2)}

/* ═══ TOOLS DARK ════════════════════════════════════════════ */
.tools-dark{background:var(--g);padding:52px 24px}
.td-hdr{max-width:var(--mw);margin:0 auto 32px}
.td-hdr h2{font-size:clamp(22px,3.5vw,32px);font-weight:900;color:#fff;letter-spacing:-.5px;margin-bottom:6px}
.td-hdr p{color:#86efac;font-size:15px}
.tools-grid{max-width:var(--mw);margin:0 auto;display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));gap:14px}
.tool-card{background:rgba(255,255,255,.08);border:1px solid rgba(255,255,255,.12);border-radius:var(--r);padding:20px;text-align:left;cursor:pointer;transition:var(--tr)}
.tool-card:hover{background:rgba(255,255,255,.15);transform:translateY(-2px)}
.tc-icon{font-size:26px;margin-bottom:10px}
.tc-name{font-family:'Fraunces',serif;font-size:16px;font-weight:700;color:#fff;margin-bottom:4px}
.tc-desc{font-size:12.5px;color:#86efac}

/* ═══ GUIDE CARD ════════════════════════════════════════════ */
.guide-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(280px,1fr));gap:16px}
.guide-card{background:#fff;border:1px solid var(--bd);border-radius:var(--r);padding:20px;cursor:pointer;transition:var(--tr)}
.guide-card:hover{border-color:var(--g);transform:translateY(-2px);box-shadow:var(--shm)}
.gc-meta{font-family:'DM Mono',monospace;font-size:10.5px;color:var(--mu);margin-bottom:10px;text-transform:uppercase;letter-spacing:1px}
.gc-title{font-family:'Fraunces',serif;font-size:16px;font-weight:700;color:var(--tx);line-height:1.3;margin-bottom:8px}
.gc-intro{font-size:13px;color:var(--mu);line-height:1.6;margin-bottom:12px;display:-webkit-box;-webkit-line-clamp:2;-webkit-box-orient:vertical;overflow:hidden}
.gc-lnk{font-size:13px;color:var(--g);font-weight:600}

/* ═══ TRUST & NEWSLETTER ════════════════════════════════════ */
.trust-strip{background:#f0fdf4;border-top:1px solid #bbf7d0;border-bottom:1px solid #bbf7d0;padding:28px 24px}
.trust-in{max-width:1100px;margin:0 auto;display:flex;flex-wrap:wrap;gap:22px;justify-content:center}
.trust-item{font-size:13.5px;color:var(--gmid);font-weight:500;display:flex;align-items:center;gap:6px}
.newsletter{background:#111827;padding:52px 24px}
.nl-in{max-width:500px;margin:0 auto;text-align:center}
.nl-in h2{font-family:'Fraunces',serif;font-size:28px;font-weight:900;color:#fff;letter-spacing:-.5px;margin-bottom:10px}
.nl-in p{color:#9ca3af;font-size:14px;margin-bottom:26px}
.nl-form{display:flex;gap:10px;flex-wrap:wrap}
.nl-form input{flex:1;min-width:0;padding:13px 16px;border-radius:8px;border:none;font-size:14px;outline:none}
.nl-form button{padding:13px 20px;background:var(--a);color:#fff;border:none;border-radius:8px;font-weight:700;font-size:14px;cursor:pointer;white-space:nowrap}
.nl-note{font-size:11px;color:#4b5563;margin-top:12px}

/* ═══ DIRECTORY ═════════════════════════════════════════════ */
.dir-search{background:#fff;border:1px solid var(--bd);border-radius:var(--r);padding:4px 4px 4px 16px;display:flex;gap:8px;margin-bottom:16px;box-shadow:var(--sh)}
.dir-search input{flex:1;border:none;outline:none;font-size:14px;color:var(--tx);background:transparent;padding:8px 0}
.dir-filters{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:16px;align-items:center}
.fsel{padding:9px 13px;border:1px solid var(--bd);border-radius:7px;font-size:13.5px;color:var(--t2);background:#fff;outline:none;cursor:pointer}
.fsel:focus{border-color:var(--g)}
.fcheck{display:flex;align-items:center;gap:7px;font-size:13.5px;color:var(--t2);cursor:pointer;padding:9px 0;white-space:nowrap}
.fcheck input{width:16px;height:16px;cursor:pointer;accent-color:var(--g)}
.rc{font-size:13.5px;color:var(--mu);margin-bottom:18px}
.rc strong{color:var(--tx)}
.no-res{text-align:center;padding:72px 24px;color:var(--mu);grid-column:1/-1}
.no-res .nr-ico{font-size:48px;margin-bottom:14px}
.no-res h3{font-family:'Fraunces',serif;font-size:22px;color:var(--t2);margin-bottom:8px}

/* ═══ DETAIL ════════════════════════════════════════════════ */
.breadcrumb{display:flex;align-items:center;gap:6px;font-size:13px;color:var(--mu);margin-bottom:24px;flex-wrap:wrap}
.breadcrumb button{background:none;border:none;color:var(--mu);cursor:pointer;font-size:13px}
.breadcrumb button:hover{color:var(--g)}
.breadcrumb .sep{color:#d1d5db}
.detail-h1{font-size:clamp(24px,4vw,40px);font-weight:900;color:var(--tx);line-height:1.15;letter-spacing:-1.5px;margin-bottom:20px}
.qf-bar{display:flex;flex-wrap:wrap;gap:14px;padding:16px 20px;background:#f0fdf4;border:1px solid #d1fae5;border-radius:var(--r);margin-bottom:26px}
.qf-item{display:flex;align-items:center;gap:6px;font-size:13px;color:var(--t2)}
.qf-item.urg{color:var(--rd);font-weight:700}
.urg-banner{border-radius:8px;padding:12px 18px;margin-bottom:22px;display:flex;align-items:center;gap:10px;font-size:13.5px;font-weight:600}
.det-sec{margin-bottom:28px;padding-bottom:28px;border-bottom:1px solid #f3f4f6}
.det-sec h2{font-family:'Fraunces',serif;font-size:19px;font-weight:700;color:var(--tx);margin-bottom:14px}
.elig-list{list-style:none;display:flex;flex-direction:column;gap:8px}
.elig-list li{font-size:14.5px;color:var(--t2);padding-left:20px;position:relative;line-height:1.6}
.elig-list li::before{content:"✓";position:absolute;left:0;color:var(--g);font-weight:700}
.doc-list{display:flex;flex-direction:column;gap:8px}
.doc-item{display:flex;gap:10px;align-items:flex-start;padding:10px 14px;background:var(--sur);border-radius:7px;border:1px solid #f3f4f6}
.doc-check{color:var(--g);font-weight:700;margin-top:1px;flex-shrink:0}
.apply-block{background:var(--g);border-radius:var(--r);padding:28px;text-align:center;margin:32px 0}
.apply-block p{color:#bbf7d0;font-size:13px;margin-bottom:16px}
.apply-btn{display:inline-block;padding:15px 40px;background:var(--a);color:#fff;border-radius:9px;font-weight:700;font-size:16px;text-decoration:none;transition:var(--tr)}
.apply-btn:hover{background:var(--ald);transform:translateY(-1px)}
.apply-note{color:#6ee7b7;font-size:11px;margin-top:14px}
.share-row{display:flex;gap:10px;flex-wrap:wrap;margin-top:16px;justify-content:center}
.share-btn{padding:9px 18px;border-radius:7px;font-size:13px;font-weight:600;cursor:pointer;border:none;display:flex;align-items:center;gap:6px;transition:var(--tr)}
.share-wa{background:#25d366;color:#fff}.share-wa:hover{background:#1da851}
.share-copy{background:#f3f4f6;color:var(--t2);border:1px solid var(--bd)}.share-copy:hover{background:#e5e7eb}

/* ═══ CALENDAR ══════════════════════════════════════════════ */
.cal-ctrl{display:flex;gap:10px;flex-wrap:wrap;margin-bottom:24px;align-items:center}
.view-toggle{display:flex;gap:4px;margin-left:auto}
.vbtn{padding:9px 18px;border:1px solid var(--bd);border-radius:7px;font-size:13px;cursor:pointer;font-weight:600;transition:var(--tr);background:#fff}
.vbtn.act{background:var(--g);color:#fff;border-color:var(--g)}
.cal-nav{display:flex;align-items:center;justify-content:space-between;margin-bottom:16px}
.cal-nav h2{font-family:'Fraunces',serif;font-size:22px;font-weight:700;color:var(--tx)}
.cnav-btn{padding:8px 16px;border:1px solid var(--bd);border-radius:7px;background:#fff;cursor:pointer;font-size:13px;font-weight:600;color:var(--t2);transition:var(--tr)}
.cnav-btn:hover{border-color:var(--g);color:var(--g)}
.cgrid{display:grid;grid-template-columns:repeat(7,1fr);gap:3px}
.cdh{text-align:center;font-size:10.5px;font-weight:700;color:var(--mu);padding:8px 0;text-transform:uppercase;letter-spacing:1px}
.cc{min-height:72px;padding:7px;border-radius:8px;border:2px solid transparent;transition:var(--tr);background:#fafafa}
.cc.has{background:#fff;border-color:var(--bd);cursor:pointer}
.cc.has:hover{border-color:var(--g);background:var(--glt)}
.cc.today{border-color:var(--a)!important;background:var(--alt)}
.cc.sel{border-color:var(--g)!important;background:#f0fdf4}
.cdn{font-size:13px;font-weight:600;color:var(--t2)}
.cc.today .cdn{color:#d97706}
.cdots{display:block;margin-top:4px;font-size:10px;font-family:'DM Mono',monospace;padding:2px 7px;border-radius:8px}
.day-panel{margin-top:20px;padding:20px;background:#f0fdf4;border:1px solid #bbf7d0;border-radius:var(--r)}
.day-panel h3{font-family:'Fraunces',serif;font-size:16px;font-weight:700;margin-bottom:14px}
.lmh{font-family:'Fraunces',serif;font-size:18px;font-weight:700;color:var(--t2);margin-bottom:12px;padding-bottom:8px;border-bottom:2px solid #f3f4f6}
.li{display:grid;grid-template-columns:120px 1fr 88px;gap:14px;padding:13px 16px;background:#fff;border:1px solid var(--bd);border-radius:9px;margin-bottom:8px;cursor:pointer;transition:var(--tr);align-items:center}
.li:hover{border-color:var(--g);background:#f8fffe}
.li-dt{font-family:'DM Mono',monospace;font-size:11px;color:var(--mu)}
.li-ttl{font-weight:600;font-size:14px;color:var(--tx);display:block;margin-bottom:2px}
.li-sub{font-size:11.5px;color:var(--mu)}
.li-days{font-family:'DM Mono',monospace;font-size:11px;font-weight:700;text-align:right}

/* ═══ CHECKER ════════════════════════════════════════════════ */
.checker-card{background:#fff;border:1px solid var(--bd);border-radius:14px;padding:28px;margin-bottom:24px}
.fl{display:block;font-size:12px;font-weight:700;color:var(--t2);margin-bottom:6px;text-transform:uppercase;letter-spacing:.8px}
.flds{width:100%;padding:10px 14px;border:1px solid var(--bd);border-radius:7px;font-size:14px;color:var(--tx);background:#fff;outline:none}
.flds:focus{border-color:var(--g)}
.rg{display:flex;gap:10px;flex-wrap:wrap}
.rb{padding:8px 22px;border-radius:7px;border:2px solid var(--bd);background:#fff;color:var(--t2);font-weight:600;font-size:13.5px;cursor:pointer;transition:var(--tr)}
.rb.sel{background:var(--g);color:#fff;border-color:var(--g)}
.risk-card{border-radius:var(--r);padding:24px;margin-bottom:20px}
.risk-card.high{background:var(--rlt);border:1px solid #fca5a5}
.risk-card.med{background:var(--alt);border:1px solid #fde68a}
.risk-card.low{background:var(--glt);border:1px solid #bbf7d0}
.risk-level{font-family:'Fraunces',serif;font-size:30px;font-weight:900;margin-bottom:10px}
.risk-advice{font-size:14.5px;color:var(--t2);font-weight:500;margin-bottom:14px;line-height:1.6}
.risk-flags{list-style:none;display:flex;flex-direction:column;gap:6px;margin-bottom:14px}
.risk-flags li{font-size:13.5px;color:var(--t2)}

/* ═══ CHECKLIST TOOL ════════════════════════════════════════ */
.cl-card{background:#fff;border:1px solid var(--bd);border-radius:var(--r);padding:20px;margin-bottom:16px}
.cl-list{list-style:none;display:flex;flex-direction:column}
.cl-item{padding:12px 0;border-bottom:1px solid #f3f4f6;display:flex;align-items:flex-start;gap:12px}
.cl-item:last-child{border-bottom:none}
.cl-item input[type=checkbox]{width:18px;height:18px;margin-top:2px;flex-shrink:0;cursor:pointer;accent-color:var(--g)}
.cl-item label{font-size:14px;color:var(--t2);cursor:pointer;line-height:1.5}
.cl-item.done label{text-decoration:line-through;color:var(--mu)}

/* ═══ BURSARY CHECKER ════════════════════════════════════════ */
.prob-result{border-radius:var(--r);padding:24px;margin-top:24px}
.prob-bar-wrap{background:rgba(0,0,0,.1);border-radius:10px;height:12px;margin:14px 0;overflow:hidden}
.prob-bar{height:100%;border-radius:10px;background:var(--a);transition:width 1s ease}

/* ═══ SAVED PAGE ════════════════════════════════════════════ */
.saved-empty{text-align:center;padding:80px 24px;color:var(--mu)}
.saved-empty .se-ico{font-size:52px;margin-bottom:16px}

/* ═══ PROVINCE PAGE ═════════════════════════════════════════ */
.prov-grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(200px,1fr));gap:12px;margin-bottom:32px}
.prov-tile{background:#fff;border:1px solid var(--bd);border-radius:var(--r);padding:18px;cursor:pointer;transition:var(--tr);text-align:left}
.prov-tile:hover{border-color:var(--g);background:var(--glt)}
.prov-tile h3{font-family:'Fraunces',serif;font-size:15px;font-weight:700;color:var(--tx);margin-bottom:4px}
.prov-tile p{font-size:12px;color:var(--mu)}

/* ═══ ABOUT ══════════════════════════════════════════════════ */
.about-accent{width:60px;height:4px;background:var(--a);border-radius:2px;margin-bottom:32px}
.acl{list-style:none;display:flex;flex-direction:column}
.acl li{padding:12px 0;border-bottom:1px solid #f3f4f6;font-size:14.5px;color:var(--t2);display:flex;gap:10px;align-items:flex-start}
.acl li::before{content:"✓";color:var(--g);font-weight:700;flex-shrink:0;margin-top:1px}
.disc-box{background:var(--rlt);border:1px solid #fca5a5;border-radius:var(--r);padding:20px;margin:32px 0}
.disc-box h3{font-family:'Fraunces',serif;font-size:16px;font-weight:700;color:var(--rd);margin-bottom:8px}
.disc-box p{font-size:13.5px;color:#7f1d1d;line-height:1.7;margin:0}

/* ═══ FOOTER ════════════════════════════════════════════════ */
footer{background:#111827;color:#9ca3af}
.fi{max-width:var(--mw);margin:0 auto;padding:52px 24px 28px;display:grid;grid-template-columns:2fr 1fr 1fr 1fr;gap:32px}
.flogo{display:flex;align-items:center;gap:8px;margin-bottom:12px}
.flogo-icon{width:30px;height:30px;background:var(--g);border-radius:6px;display:flex;align-items:center;justify-content:center;font-size:15px}
.flogo-text{font-family:'Fraunces',serif;font-size:17px;font-weight:900;color:#fff}
.fa{font-size:12.5px;color:#6b7280;line-height:1.7;margin-bottom:12px}
.ft{font-size:12px;color:#4b5563;line-height:1.9}
.fcol h4{color:#fff;font-size:11px;text-transform:uppercase;letter-spacing:1px;margin-bottom:12px;font-weight:700}
.fl-btn{display:block;background:none;border:none;color:#6b7280;font-size:13px;padding:3.5px 0;cursor:pointer;text-align:left;transition:color .15s}
.fl-btn:hover{color:#fff}
.fbar{border-top:1px solid #1f2937;max-width:var(--mw);margin:0 auto;padding:16px 24px;display:flex;justify-content:space-between;flex-wrap:wrap;gap:8px;font-size:12px;color:#4b5563}

/* ═══ RESPONSIVE ════════════════════════════════════════════ */
@media(max-width:900px){
  .fi{grid-template-columns:1fr 1fr}
  .opp-grid,.guide-grid,.cat-grid{grid-template-columns:repeat(auto-fill,minmax(260px,1fr))}
}
@media(max-width:768px){
  .nav-links{display:none}
  .hbg{display:block}
  .search-bar{flex-direction:column}
  .search-bar select{border-left:none;border-top:1px solid var(--bd);padding:10px 14px}
  .li{grid-template-columns:1fr;gap:4px}
  .li-days{text-align:left}
  .sec-hdr{flex-direction:column;gap:6px}
  .dir-filters{gap:6px}
}
@media(max-width:540px){
  .opp-grid,.guide-grid{grid-template-columns:1fr}
  .cat-grid{grid-template-columns:1fr 1fr}
  .nl-form{flex-direction:column}
  .fi{grid-template-columns:1fr}
  .stats-in{gap:18px}
  .stat-num{font-size:22px}
}
@media print{
  #nav,footer,.apply-block,.share-row,.btn,#breadcrumb{display:none}
  body{background:#fff;font-size:13px}
  .opp-card{break-inside:avoid}
}
</style>
</head>
<body>

<!-- ═══ NAV ══════════════════════════════════════════════════ -->
<nav id="nav">
  <div class="nav-in">
    <button class="logo" onclick="go('home')">
      <div class="logo-icon">🧭</div>
      <span class="logo-text">Opportunities<em>ZA</em></span>
    </button>
    <div class="nav-links">
      <button class="nl" onclick="goDir('learnership')">Learnerships</button>
      <button class="nl" onclick="goDir('bursary')">Bursaries</button>
      <button class="nl" onclick="goDir('internship')">Internships</button>
      <button class="nl" onclick="go('guides')">SETA Guides</button>
      <button class="nl" onclick="go('calendar')">📅 Calendar</button>
      <button class="nl" onclick="go('saved')">🔖 Saved</button>
      <button class="nl nl-cta" onclick="go('directory')">Browse All</button>
    </div>
    <button class="hbg" id="hbg" onclick="toggleDrw()">☰</button>
  </div>
  <div class="drawer" id="drw">
    <button class="dl" onclick="goDir('learnership');closeDrw()">🎓 Learnerships</button>
    <button class="dl" onclick="goDir('bursary');closeDrw()">📚 Bursaries</button>
    <button class="dl" onclick="goDir('internship');closeDrw()">💼 Internships</button>
    <button class="dl" onclick="goDir('apprenticeship');closeDrw()">🔧 Apprenticeships</button>
    <button class="dl" onclick="go('guides');closeDrw()">📖 SETA Guides</button>
    <button class="dl" onclick="go('calendar');closeDrw()">📅 Deadline Calendar</button>
    <button class="dl" onclick="go('tools');closeDrw()">🛠️ Free Tools</button>
    <button class="dl" onclick="go('saved');closeDrw()">🔖 Saved Opportunities</button>
    <button class="dl" onclick="go('provinces');closeDrw()">📍 Browse by Province</button>
    <button class="dl" onclick="go('about');closeDrw()">About Us</button>
    <button class="btn btn-primary btn-full" style="margin-top:8px;border-radius:8px;padding:12px" onclick="go('directory');closeDrw()">Browse All Opportunities</button>
  </div>
</nav>

<!-- ═══ MAIN ══════════════════════════════════════════════════ -->
<main id="app"></main>

<!-- ═══ FOOTER ═══════════════════════════════════════════════ -->
<footer>
  <div class="fi">
    <div>
      <div class="flogo"><div class="flogo-icon">🧭</div><span class="flogo-text">OpportunitiesZA</span></div>
      <p class="fa">South Africa's trusted guide to learnerships, bursaries, internships, and SETA opportunities. Verified. Free. Updated daily.</p>
      <div class="ft">✓ Never collect your documents<br>✓ Never charge application fees<br>✓ All listings link to official sources</div>
    </div>
    <div>
      <h4 class="fcol">Opportunities</h4>
      <button class="fl-btn" onclick="go('directory')">Browse All</button>
      <button class="fl-btn" onclick="goDir('learnership')">Learnerships</button>
      <button class="fl-btn" onclick="goDir('bursary')">Bursaries</button>
      <button class="fl-btn" onclick="goDir('internship')">Internships</button>
      <button class="fl-btn" onclick="goDir('apprenticeship')">Apprenticeships</button>
      <button class="fl-btn" onclick="go('provinces')">By Province</button>
    </div>
    <div>
      <h4 class="fcol">Tools & Guides</h4>
      <button class="fl-btn" onclick="go('calendar')">Deadline Calendar</button>
      <button class="fl-btn" onclick="go('eligibility')">Eligibility Checker</button>
      <button class="fl-btn" onclick="go('scam')">Scam Checker</button>
      <button class="fl-btn" onclick="go('checklist')">Checklist Generator</button>
      <button class="fl-btn" onclick="go('bursary-checker')">Bursary Checker</button>
      <button class="fl-btn" onclick="go('guides')">SETA Guides</button>
    </div>
    <div>
      <h4 class="fcol">Company</h4>
      <button class="fl-btn" onclick="go('about')">About Us</button>
      <button class="fl-btn" onclick="go('about')">Contact</button>
      <button class="fl-btn" onclick="go('about')">Disclaimer</button>
      <button class="fl-btn" onclick="go('about')">Privacy Policy</button>
    </div>
  </div>
  <div class="fbar">
    <span>© 2026 OpportunitiesZA. For information purposes only. Not an employer or recruitment agency.</span>
    <span>South Africa 🇿🇦</span>
  </div>
</footer>

<script>
// ============================================
// CONFIGURATION & ABSTRACTION LAYER
// TEMPORARY: All features use static/scaffolding implementations
// TARGET: Swap CONFIG.features values to migrate to backend
// MIGRATION: Change CONFIG.features values, implementations auto-switch
// ============================================

const CONFIG = {
  mode: 'static',        // 'static' → 'api' → 'ssr'
  features: {
    hosting: 'github',   // 'github' → 'vercel' → 'aws'
    data: 'json',        // 'json' → 'database' → 'cms'
    bookmarks: 'local',  // 'local' → 'api' → 'sync'
    search: 'client',    // 'client' → 'server' → 'engine'
    routing: 'hash',     // 'hash' → 'history' → 'ssg'
    auth: 'none',        // 'none' → 'jwt' → 'oauth'
    crawler: 'manual',   // 'manual' → 'scheduled' → 'ai'
    publishing: 'git'    // 'git' → 'api' → 'auto'
  }
};

// Storage Abstraction
// TEMPORARY: localStorage implementation
// TARGET: Cloud-synced via API
const Storage = {
  async getBookmarks() {
    if (CONFIG.features.bookmarks === 'local') {
      return JSON.parse(localStorage.getItem('careerhub_bookmarks') || '[]');
    }
    const res = await fetch('/api/bookmarks');
    return res.json();
  },
  async setBookmarks(bookmarks) {
    if (CONFIG.features.bookmarks === 'local') {
      localStorage.setItem('careerhub_bookmarks', JSON.stringify(bookmarks));
      return;
    }
    await fetch('/api/bookmarks', {method: 'POST', body: JSON.stringify(bookmarks)});
  }
};

// API Abstraction
// TEMPORARY: Static JSON files
// TARGET: REST API endpoints
const API = {
  async getOpportunities() {
    if (CONFIG.features.data === 'json') {
      const res = await fetch('/data/opportunities.json');
      return res.json();
    }
    const res = await fetch('/api/opportunities');
    return res.json();
  },
  async getGuides() {
    if (CONFIG.features.data === 'json') {
      const res = await fetch('/data/guides.json');
      return res.json();
    }
    const res = await fetch('/api/guides');
    return res.json();
  },
  async getCategories() {
    if (CONFIG.features.data === 'json') {
      const res = await fetch('/data/categories.json');
      return res.json();
    }
    const res = await fetch('/api/categories');
    return res.json();
  },
  async getProvinces() {
    if (CONFIG.features.data === 'json') {
      const res = await fetch('/data/provinces.json');
      return res.json();
    }
    const res = await fetch('/api/provinces');
    return res.json();
  }
};

// Search Abstraction
// TEMPORARY: Client-side filtering
// TARGET: Server-side search (Algolia/Typesense)
const Search = {
  async search(query, filters) {
    if (CONFIG.features.search === 'client') {
      const data = await API.getOpportunities();
      // client-side filtering logic
      let results = data;
      if (query) {
        const q = query.toLowerCase();
        results = results.filter(op =>
          op.title.toLowerCase().includes(q) ||
          op.description.toLowerCase().includes(q)
        );
      }
      if (filters.province) {
        results = results.filter(op => op.province === filters.province);
      }
      if (filters.category) {
        results = results.filter(op => op.category === filters.category);
      }
      return results;
    }
    const params = new URLSearchParams({q: query, ...filters});
    const res = await fetch(`/api/search?${params}`);
    return res.json();
  }
};

// Router Abstraction
// TEMPORARY: Hash-based routing
// TARGET: History API with SSR
const Router = {
  navigate(path) {
    const route = path.startsWith('/') ? path : `/${path}`;
    if (CONFIG.features.routing === 'hash') {
      window.location.hash = route;
    } else {
      history.pushState({}, '', route);
      window.dispatchEvent(new PopStateEvent('popstate'));
    }
  },
  getCurrentRoute() {
    if (CONFIG.features.routing === 'hash') {
      return window.location.hash.slice(1) || '/';
    }
    return window.location.pathname;
  }
};
// ============================================
// END ABSTRACTION LAYER
// ============================================

/* ═══════════════════════════════════════════════════════════
   DATA — LOADED FROM /data/*.json
═══════════════════════════════════════════════════════════ */
const DATA_BASE='/data';
let OPPS=[];
let GUIDES=[];
let CATS=[];
let PROVS=[];

async function fetchJSON(file){
  const res=await fetch(`${DATA_BASE}/${file}`,{cache:'no-store'});
  if(!res.ok) throw new Error(`${file} failed with ${res.status}`);
  return res.json();
}

function mapOpportunity(o){
  return{
    id:String(o.id),
    title:o.title,
    cat:o.category,
    sector:o.sector||'',
    prov:o.province,
    qual:o.qualification_level,
    stipend:o.stipend||'',
    close:o.closing_date,
    url:o.application_url,
    src:o.source_name||'Official source',
    srcUrl:o.source_url||o.application_url,
    verified:Boolean(o.verified),
    featured:Boolean(o.featured),
    expired:Boolean(o.expired),
    slug:o.slug,
    desc:o.description||'',
    elig:o.eligibility||[],
    docs:o.documents_needed||[],
    tags:o.tags||[]
  };
}

function mapGuide(g){
  return{
    id:String(g.id),
    title:g.title,
    slug:g.slug,
    cat:g.category,
    intro:g.intro||'',
    body:g.body||'',
    read:g.read_time||'4 min'
  };
}

function mapCategory(c){
  return{
    id:c.id,
    label:c.label,
    icon:c.icon||'•',
    desc:c.description||c.desc||'',
    color:c.color||'#374151',
    bg:c.bg||'#f3f4f6'
  };
}

function mapProvinces(provinces){
  return['Nationwide',...provinces.map(p=>p.label)];
}

async function loadContent(){
  const [opps,guides,cats,provinces]=await Promise.all([
    fetchJSON('opportunities.json'),
    fetchJSON('guides.json'),
    fetchJSON('categories.json'),
    fetchJSON('provinces.json')
  ]);
  OPPS=opps.map(mapOpportunity);
  GUIDES=guides.map(mapGuide);
  CATS=cats.map(mapCategory);
  PROVS=mapProvinces(provinces);
}

function renderLoading(){
  const app=document.getElementById('app');
  if(app) app.innerHTML=`<div class="page" style="max-width:var(--mw);margin:0 auto;padding:70px 24px;text-align:center">
    <h1 style="font-family:'Fraunces',serif;color:var(--g);font-size:32px;margin-bottom:10px">Loading opportunities…</h1>
    <p style="color:var(--mu)">Fetching the latest listings and guides.</p>
  </div>`;
}

function renderLoadError(err){
  const app=document.getElementById('app');
  if(app) app.innerHTML=`<div class="page" style="max-width:var(--cw);margin:0 auto;padding:70px 24px;text-align:center">
    <h1 style="font-family:'Fraunces',serif;color:var(--rd);font-size:30px;margin-bottom:10px">We could not load the listings</h1>
    <p style="color:var(--t2);margin-bottom:22px">Please refresh the page. If the problem continues, check that the content files are available in <code>/data/</code>.</p>
    <pre style="white-space:pre-wrap;text-align:left;background:#fff;border:1px solid var(--bd);border-radius:var(--r);padding:14px;color:var(--mu);font-size:12px">${err.message}</pre>
  </div>`;
}

/* ═══════════════════════════════════════════════════════════
   UTILS
═══════════════════════════════════════════════════════════ */
const days=d=>Math.ceil((new Date(d)-new Date())/86400000);
const fmtD=d=>new Date(d).toLocaleDateString("en-ZA",{day:"numeric",month:"long",year:"numeric"});
const fmtMY=d=>new Date(d).toLocaleDateString("en-ZA",{month:"long",year:"numeric"});
const fmtShort=d=>new Date(d).toLocaleDateString("en-ZA",{day:"numeric",month:"short"});

function urgInfo(d){
  const n=days(d);
  if(n<0) return{label:"Closed",cls:"",c:"#6b7280",bg:"#f3f4f6"};
  if(n<=3) return{label:`${n}d left`,cls:"pill-red",c:"#dc2626",bg:"#fee2e2"};
  if(n<=7) return{label:`${n}d left`,cls:"pill-amber",c:"#d97706",bg:"#fef3c7"};
  if(n<=30) return{label:`${n}d left`,cls:"pill-green",c:"#166534",bg:"#dcfce7"};
  return{label:fmtD(d),cls:"pill-gray",c:"#374151",bg:"#f3f4f6"};
}
const catInfo=id=>CATS.find(c=>c.id===id)||{color:"#374151",bg:"#f3f4f6"};
const badge=cat=>`<span class="badge badge-${cat}">${cat}</span>`;
const pill=(d)=>{const u=urgInfo(d);return`<span class="pill ${u.cls}" style="background:${u.bg};color:${u.c}">${u.label}</span>`;};
const liveOpps=()=>OPPS.filter(o=>!o.expired&&days(o.close)>=0);

// Saved (localStorage)
const getSaved=()=>{try{return JSON.parse(localStorage.getItem('opza_saved')||'[]')}catch{return[]}};
const isSaved=id=>getSaved().includes(id);
function toggleSave(id,e){
  e&&e.stopPropagation();
  const s=getSaved();
  const idx=s.indexOf(id);
  if(idx>-1) s.splice(idx,1); else s.push(id);
  localStorage.setItem('opza_saved',JSON.stringify(s));
  document.querySelectorAll(`.bk-${id}`).forEach(el=>el.textContent=isSaved(id)?'🔖':'🔲');
}
const bkBtn=id=>`<button class="bk-btn bk-${id}" onclick="toggleSave('${id}',event)" title="${isSaved(id)?'Remove bookmark':'Save opportunity'}">${isSaved(id)?'🔖':'🔲'}</button>`;

/* ═══════════════════════════════════════════════════════════
   ROUTER
═══════════════════════════════════════════════════════════ */
let _page='home',_opp=null,_guide=null,_dirCat='';
let calState={year:new Date().getFullYear(),month:new Date().getMonth(),view:'list',cat:'',sel:null};

class RouterWrapper {
  constructor(router){
    this.router=router;
  }
  normalize(path){
    return (path||'/').startsWith('/') ? path : `/${path}`;
  }
  getCurrentRoute(){
    return this.normalize(this.router.getCurrentRoute());
  }
  navigate(path){
    const route=this.normalize(path);
    if(this.getCurrentRoute()===route){
      this.apply(route);
      return;
    }
    this.router.navigate(route);
  }
  pageToPath(page,data=null){
    if(page==='home') return '/';
    if(page==='directory'&&typeof data==='string'&&data) return `/directory/${encodeURIComponent(data)}`;
    if(page==='detail') return `/opportunity/${encodeURIComponent((data&&data.id)||(_opp&&_opp.id)||'')}`;
    if(page==='guide-detail') return `/guides/${encodeURIComponent((data&&data.id)||(_guide&&_guide.id)||'')}`;
    return `/${page}`;
  }
  apply(route=this.getCurrentRoute()){
    const [path]=this.normalize(route).split('?');
    const parts=path.split('/').filter(Boolean);
    _dirCat='';
    if(parts.length===0){
      _page='home';
    }else if(parts[0]==='directory'){
      _page='directory';
      _dirCat=decodeURIComponent(parts[1]||'');
    }else if(parts[0]==='opportunity'){
      _page='detail';
      _opp=OPPS.find(o=>o.id===decodeURIComponent(parts[1]||''))||_opp;
    }else if(parts[0]==='guides'&&parts[1]){
      _page='guide-detail';
      _guide=GUIDES.find(g=>g.id===decodeURIComponent(parts[1]))||_guide;
    }else{
      _page=parts[0];
    }
    window.scrollTo(0,0);
    closeDrw();
    render();
  }
  start(){
    this.apply(this.getCurrentRoute());
  }
}

const AppRouter=new RouterWrapper(Router);
window.addEventListener('hashchange',()=>AppRouter.apply(AppRouter.getCurrentRoute()));
if(CONFIG.features.routing!=='hash') window.addEventListener('popstate',()=>AppRouter.apply(AppRouter.getCurrentRoute()));

function go(p,data=null){
  if(data&&typeof data==='object'){if(p==='detail')_opp=data;if(p==='guide-detail')_guide=data;}
  if(p==='directory'&&typeof data==='string') _dirCat=data;
  AppRouter.navigate(AppRouter.pageToPath(p,data));
}
function goDir(cat){go('directory',cat);}
function openOpp(id){const opp=OPPS.find(o=>o.id===id);if(opp)go('detail',opp);}
function openGuide(id){const guide=GUIDES.find(g=>g.id===id);if(guide)go('guide-detail',guide);}

function render(){
  const app=document.getElementById('app');
  switch(_page){
    case 'home': app.innerHTML=renderHome();break;
    case 'directory': app.innerHTML=renderDir(_dirCat);bindDir();break;
    case 'detail': app.innerHTML=renderDetail();break;
    case 'calendar': app.innerHTML=renderCal();break;
    case 'tools': app.innerHTML=renderTools();break;
    case 'eligibility': app.innerHTML=renderElig();break;
    case 'scam': app.innerHTML=renderScam();break;
    case 'checklist': app.innerHTML=renderChecklist();break;
    case 'bursary-checker': app.innerHTML=renderBursaryChecker();break;
    case 'guides': app.innerHTML=renderGuides();break;
    case 'guide-detail': app.innerHTML=renderGuideDetail();break;
    case 'provinces': app.innerHTML=renderProvinces();break;
    case 'saved': app.innerHTML=renderSaved();break;
    case 'about': app.innerHTML=renderAbout();break;
    default: app.innerHTML=renderHome();
  }
}

/* ═══════════════════════════════════════════════════════════
   NAV HELPERS
═══════════════════════════════════════════════════════════ */
function toggleDrw(){const d=document.getElementById('drw'),h=document.getElementById('hbg');d.classList.toggle('open');h.textContent=d.classList.contains('open')?'✕':'☰';}
function closeDrw(){const d=document.getElementById('drw'),h=document.getElementById('hbg');d.classList.remove('open');if(h)h.textContent='☰';}
window.addEventListener('scroll',()=>document.getElementById('nav').classList.toggle('scrolled',scrollY>20));

/* ═══════════════════════════════════════════════════════════
   OPP CARD
═══════════════════════════════════════════════════════════ */
function oppCard(o,compact=false){
  const u=urgInfo(o.close);
  return`<div class="opp-card${o.featured?' featured':''}" onclick="openOpp('${o.id}')">
  <div class="oc-top">${badge(o.cat)}${o.verified?'<span class="oc-verified">✓ Verified</span>':''}</div>
  <h3 class="oc-title">${o.title}</h3>
  <div class="oc-meta"><span>📍 ${o.prov}</span>${o.stipend?`<span>💰 ${o.stipend}</span>`:''}</div>
  <div class="oc-footer">${pill(o.close)}<div style="display:flex;align-items:center;gap:10px"><span class="oc-link">View & Apply →</span>${bkBtn(o.id)}</div></div>
  </div>`;
}

/* ═══════════════════════════════════════════════════════════
   HOME
═══════════════════════════════════════════════════════════ */
function renderHome(){
  const live=liveOpps();
  const closing=[...live].filter(o=>days(o.close)<=30).sort((a,b)=>days(a.close)-days(b.close)).slice(0,7);
  const featured=[...live].sort((a,b)=>new Date(a.close)-new Date(b.close)).slice(0,6);
  return`<div class="page">
  <!-- HERO -->
  <div class="hero">
    <div class="hero-in">
      <div class="hero-badge"><span>✓ Verified · Free · South Africa Only</span></div>
      <h1>Find Your <em>Career</em><br>Opportunity in SA</h1>
      <p class="hero-sub">Learnerships, bursaries, internships, and apprenticeships — verified against official sources and updated daily.</p>
      <div class="search-bar">
        <input id="hq" placeholder="Search learnerships, bursaries, companies..." onkeydown="if(event.key==='Enter')heroSearch()">
        <select id="hc">
          <option value="">All Types</option>
          ${CATS.map(c=>`<option value="${c.id}">${c.label}</option>`).join('')}
        </select>
        <button onclick="heroSearch()">Search →</button>
      </div>
      <div class="chips">
        ${['Learnerships','Bursaries','Internships','Closing Soon','By Province'].map(c=>`
          <button class="chip" onclick="${c==='Closing Soon'?"go('calendar')":c==='By Province'?"go('provinces')":`goDir('${c.toLowerCase().slice(0,-1)}')`}">${c}</button>
        `).join('')}
      </div>
    </div>
  </div>

  <!-- STATS BAR -->
  <div class="stats-bar">
    <div class="stats-in">
      <div class="stat"><span class="stat-num">${live.length}</span><span class="stat-label">Open Listings</span></div>
      <div class="stat"><span class="stat-num">${live.filter(o=>days(o.close)<=30).length}</span><span class="stat-label">Closing This Month</span></div>
      <div class="stat"><span class="stat-num">21</span><span class="stat-label">SETAs Covered</span></div>
      <div class="stat"><span class="stat-num">9</span><span class="stat-label">Provinces</span></div>
      <div class="stat"><span class="stat-num">100%</span><span class="stat-label">Free to Apply</span></div>
    </div>
  </div>

  <!-- CATEGORIES -->
  <div style="padding:52px 24px 0;max-width:var(--mw);margin:0 auto">
    <div class="cat-grid">
      ${CATS.map(c=>{const cnt=live.filter(o=>o.cat===c.id).length;return`
        <button class="cat-tile" onclick="goDir('${c.id}')">
          <div class="ct-icon">${c.icon}</div>
          <div class="ct-name">${c.label}</div>
          <div class="ct-desc">${c.desc}</div>
          <div class="ct-count" style="color:${c.color};background:${c.bg}">${cnt} open now</div>
        </button>`;}).join('')}
    </div>
  </div>

  <!-- CLOSING STRIP -->
  ${closing.length?`<div class="strip" style="margin-top:48px">
    <div class="strip-in">
      <div class="strip-hdr"><span>🚨</span><span>Closing Soon — Don't Miss These</span></div>
      <div class="strip-scroll">
        ${closing.map(o=>{const u=urgInfo(o.close);return`
          <button class="strip-pill" onclick="openOpp('${o.id}')">
            <span class="sp-title">${o.title}</span>
            <span class="sp-meta">📍 ${o.prov} · ${badge(o.cat).replace(/<[^>]+>/g,'').trim()}</span>
            <span class="pill ${u.cls}" style="background:${u.bg};color:${u.c};font-size:10px">${u.label}</span>
          </button>`;}).join('')}
      </div>
    </div>
  </div>`:''}

  <!-- FEATURED OPPS -->
  <div style="max-width:var(--mw);margin:0 auto;padding:52px 24px 0">
    <div class="sec-hdr">
      <h2 class="sec-title">Latest Opportunities</h2>
      <button class="sec-lnk" onclick="go('directory')">View all ${live.length} →</button>
    </div>
    <div class="opp-grid">${featured.map(o=>oppCard(o)).join('')}</div>
  </div>

  <!-- TOOLS -->
  <div class="tools-dark" style="margin-top:60px">
    <div class="td-hdr">
      <h2>Apply Smarter with Free Tools</h2>
      <p>No login, no fees, nothing collected — built for South African job seekers</p>
    </div>
    <div class="tools-grid">
      ${[
        {icon:"✅",name:"Eligibility Checker",desc:"Find what you qualify for",p:"eligibility"},
        {icon:"📅",name:"Deadline Calendar",desc:"Never miss a closing date",p:"calendar"},
        {icon:"🛡️",name:"Scam Checker",desc:"Verify before you apply",p:"scam"},
        {icon:"📋",name:"Checklist Generator",desc:"Documents you need to apply",p:"checklist"},
        {icon:"🎯",name:"Bursary Probability",desc:"How likely are you to qualify?",p:"bursary-checker"},
        {icon:"🗺️",name:"Browse by Province",desc:"Opportunities near you",p:"provinces"},
      ].map(t=>`<button class="tool-card" onclick="go('${t.p}')">
        <div class="tc-icon">${t.icon}</div>
        <div class="tc-name">${t.name}</div>
        <div class="tc-desc">${t.desc}</div>
      </button>`).join('')}
    </div>
  </div>

  <!-- GUIDES -->
  <div style="max-width:var(--mw);margin:0 auto;padding:52px 24px 0">
    <div class="sec-hdr">
      <h2 class="sec-title">Understand the System</h2>
      <button class="sec-lnk" onclick="go('guides')">All ${GUIDES.length} guides →</button>
    </div>
    <div class="guide-grid">
      ${GUIDES.slice(0,3).map(g=>`
        <div class="guide-card" onclick="openGuide('${g.id}')">
          <div class="gc-meta">📖 ${g.cat.replace('-',' ')} · ${g.read}</div>
          <h3 class="gc-title">${g.title}</h3>
          <p class="gc-intro">${g.intro}</p>
          <span class="gc-lnk">Read guide →</span>
        </div>`).join('')}
    </div>
  </div>

  <!-- TRUST -->
  <div class="trust-strip" style="margin-top:52px">
    <div class="trust-in">
      ${["✓ All sources verified against official websites","✓ Updated daily","✓ South Africa only","✓ We never collect your documents","✓ Zero application fees","✓ Scam-free directory"].map(t=>`<span class="trust-item">${t}</span>`).join('')}
    </div>
  </div>

  <!-- NEWSLETTER -->
  <div class="newsletter">
    <div class="nl-in">
      <h2>Get Daily Opportunity Alerts</h2>
      <p>Join South Africans who never miss a closing date. Free, no spam, unsubscribe anytime.</p>
      <div class="nl-form">
        <input type="email" placeholder="Your email address">
        <button onclick="alert('Subscribed! You\'ll receive your first update shortly. ✓')">Subscribe Free</button>
      </div>
      <p class="nl-note">Your details are never shared. Unsubscribe anytime.</p>
    </div>
  </div>
  </div>`;
}

function heroSearch(){
  const q=document.getElementById('hq')?.value||'';
  const c=document.getElementById('hc')?.value||'';
  _dirCat=c;
  go('directory');
  setTimeout(()=>{
    const si=document.getElementById('di');
    const sc=document.getElementById('dc');
    if(si)si.value=q;
    if(sc&&c)sc.value=c;
    applyDir();
  },60);
}

/* ═══════════════════════════════════════════════════════════
   DIRECTORY
═══════════════════════════════════════════════════════════ */
function renderDir(defCat=''){
  const live=liveOpps();
  const res=filterOpps(live,defCat,'','',false).sort((a,b)=>new Date(a.close)-new Date(b.close));
  return`<div class="page" style="max-width:var(--mw);margin:0 auto;padding:40px 24px 80px">
  <h1 style="font-size:clamp(26px,5vw,44px);font-weight:900;color:var(--g);letter-spacing:-1.5px;margin-bottom:8px">Browse Opportunities</h1>
  <p style="color:var(--mu);margin-bottom:24px;font-size:15px">Searching ${live.length} verified listings across South Africa</p>

  <div class="dir-search">
    <input id="di" placeholder="Search by company, sector, keyword..." oninput="applyDir()">
    <button class="btn btn-ghost btn-sm" onclick="document.getElementById('di').value='';applyDir()">Clear</button>
  </div>
  <div class="dir-filters">
    <select class="fsel" id="dc" onchange="applyDir()">
      <option value="">All Categories</option>
      ${CATS.map(c=>`<option value="${c.id}" ${c.id===defCat?'selected':''}>${c.label}</option>`).join('')}
    </select>
    <select class="fsel" id="dp" onchange="applyDir()">
      <option value="">All Provinces</option>
      ${PROVS.filter(p=>p!=='Nationwide').map(p=>`<option value="${p}">${p}</option>`).join('')}
    </select>
    <select class="fsel" id="dq" onchange="applyDir()">
      <option value="">Any Qualification</option>
      <option value="Matric">Matric</option>
      <option value="Diploma">Diploma</option>
      <option value="Degree">Degree</option>
    </select>
    <select class="fsel" id="ds" onchange="applyDir()">
      <option value="date">Sort: Closing Date</option>
      <option value="urgent">Sort: Most Urgent</option>
    </select>
    <label class="fcheck"><input type="checkbox" id="dv" onchange="applyDir()"> Verified only</label>
    <button class="btn btn-ghost btn-sm" onclick="resetDir()">Clear All</button>
  </div>
  <div id="drc" class="rc"><strong>${res.length}</strong> opportunities found</div>
  <div id="dr" class="opp-grid">${res.length?res.map(o=>oppCard(o)).join(''):noRes()}</div>
  </div>`;
}

function filterOpps(list,cat,prov,q,verOnly){
  return list.filter(o=>{
    if(cat&&o.cat!==cat) return false;
    if(prov&&o.prov!==prov&&o.prov!=='Nationwide') return false;
    if(verOnly&&!o.verified) return false;
    if(q){const lq=q.toLowerCase();if(!o.title.toLowerCase().includes(lq)&&!o.sector.toLowerCase().includes(lq)&&!o.prov.toLowerCase().includes(lq)) return false;}
    return true;
  });
}

function applyDir(){
  const cat=document.getElementById('dc')?.value||'';
  const prov=document.getElementById('dp')?.value||'';
  const qual=document.getElementById('dq')?.value||'';
  const sort=document.getElementById('ds')?.value||'date';
  const q=document.getElementById('di')?.value||'';
  const ver=document.getElementById('dv')?.checked||false;
  let live=liveOpps();
  let res=filterOpps(live,cat,prov,q,ver);
  if(qual) res=res.filter(o=>o.qual===qual);
  if(sort==='date') res=res.sort((a,b)=>new Date(a.close)-new Date(b.close));
  if(sort==='urgent') res=res.sort((a,b)=>days(a.close)-days(b.close));
  const rc=document.getElementById('drc'),dr=document.getElementById('dr');
  if(rc)rc.innerHTML=`<strong>${res.length}</strong> opportunities found`;
  if(dr)dr.innerHTML=res.length?res.map(o=>oppCard(o)).join(''):noRes();
}

function resetDir(){
  ['dc','dp','dq','ds'].forEach(id=>{const e=document.getElementById(id);if(e)e.value=id==='ds'?'date':'';});
  const di=document.getElementById('di');if(di)di.value='';
  const dv=document.getElementById('dv');if(dv)dv.checked=false;
  applyDir();
}

function noRes(){return`<div class="no-res"><div class="nr-ico">🔍</div><h3>No results found</h3><p style="font-size:14px;margin-bottom:20px">Try adjusting filters or search term</p><button class="btn btn-primary" onclick="resetDir()">Clear all filters</button></div>`;}
function bindDir(){}

/* ═══════════════════════════════════════════════════════════
   DETAIL
═══════════════════════════════════════════════════════════ */
function renderDetail(){
  const o=_opp;
  if(!o)return'<div style="padding:60px;text-align:center">Not found.</div>';
  const u=urgInfo(o.close);
  const rel=liveOpps().filter(r=>r.cat===o.cat&&r.id!==o.id).slice(0,3);
  const waText=encodeURIComponent(`🚨 ${o.title}\n📍 ${o.prov}\n💰 ${o.stipend}\n⏳ Closes ${fmtD(o.close)}\n\nApply here: ${o.url}`);
  return`<div class="page" style="max-width:var(--cw);margin:0 auto;padding:40px 24px 80px">
  <div class="breadcrumb">
    <button onclick="go('home')">Home</button><span class="sep">›</span>
    <button onclick="goDir('${o.cat}')">${o.cat}s</button><span class="sep">›</span>
    <span style="color:var(--t2)">${o.title}</span>
  </div>

  <div style="display:flex;gap:10px;align-items:center;flex-wrap:wrap;margin-bottom:16px">
    ${badge(o.cat)}${o.verified?'<span style="font-size:12px;color:var(--gmid);font-weight:600">✓ Verified Source</span>':''}
  </div>

  <h1 class="detail-h1">${o.title}</h1>

  <div class="qf-bar">
    <div class="qf-item">📍 ${o.prov}</div>
    <div class="qf-item${days(o.close)<=7?' urg':''}">⏳ Closes ${fmtD(o.close)}</div>
    ${o.stipend?`<div class="qf-item">💰 ${o.stipend}</div>`:''}
    <div class="qf-item">🎓 ${o.qual}</div>
    <div class="qf-item">🏢 ${o.sector}</div>
  </div>

  <div class="urg-banner" style="background:${u.bg};border:1px solid ${u.c}44;color:${u.c}">
    <span style="font-size:16px">⏳</span>
    <span>${days(o.close)<=7?`Only ${days(o.close)} days left to apply!`:`Closing: ${fmtD(o.close)}`}</span>
    ${bkBtn(o.id)}
  </div>

  <div class="det-sec">
    <h2>About This Opportunity</h2>
    <p style="font-size:15px;color:var(--t2);line-height:1.85">${o.desc}</p>
  </div>

  <div class="det-sec">
    <h2>Who Can Apply</h2>
    <ul class="elig-list">${o.elig.map(e=>`<li>${e}</li>`).join('')}</ul>
  </div>

  <div class="det-sec">
    <h2>Documents Needed</h2>
    <div class="doc-list">${o.docs.map(d=>`<div class="doc-item"><span class="doc-check">✓</span><span style="font-size:14px;color:var(--t2)">${d}</span></div>`).join('')}</div>
    <button class="btn btn-ghost btn-sm" onclick="go('checklist')" style="margin-top:12px">📋 Generate a personalised checklist →</button>
  </div>

  <div class="apply-block">
    <p>Always apply via the official company website. This links directly to the official source.</p>
    <a href="${o.url}" target="_blank" rel="noopener noreferrer" class="apply-btn">Apply Now — Official Link ↗</a>
    <div class="share-row">
      <a href="https://wa.me/?text=${waText}" target="_blank" class="share-btn share-wa">📲 Share on WhatsApp</a>
      <button class="share-btn share-copy" onclick="navigator.clipboard.writeText('${o.url}').then(()=>this.textContent='✓ Copied!')">📋 Copy Link</button>
    </div>
    <div class="apply-note">Source: ${o.src} · Last verified: ${new Date().toLocaleDateString("en-ZA")}</div>
  </div>

  ${rel.length?`<h3 style="font-family:'Fraunces',serif;font-size:22px;font-weight:700;color:var(--g);margin-bottom:16px">More ${o.cat}s</h3>
  <div class="opp-grid">${rel.map(r=>oppCard(r,true)).join('')}</div>`:''}
  </div>`;
}

/* ═══════════════════════════════════════════════════════════
   CALENDAR
═══════════════════════════════════════════════════════════ */
const MONTHS=["January","February","March","April","May","June","July","August","September","October","November","December"];
const WDAYS=["Sun","Mon","Tue","Wed","Thu","Fri","Sat"];

function renderCal(){
  return`<div class="page" style="max-width:1000px;margin:0 auto;padding:40px 24px 80px">
  <h1 style="font-size:clamp(26px,5vw,42px);font-weight:900;color:var(--g);letter-spacing:-1px;margin-bottom:8px">Deadline Calendar</h1>
  <p style="color:var(--mu);margin-bottom:24px">Never miss a closing date — all upcoming deadlines in one view</p>
  <div class="cal-ctrl">
    <select class="fsel" id="calCat" onchange="calCat(this.value)">
      <option value="">All Categories</option>
      ${CATS.map(c=>`<option value="${c.id}" ${c.id===calState.cat?'selected':''}>${c.label}</option>`).join('')}
    </select>
    <div class="view-toggle">
      <button class="vbtn ${calState.view==='list'?'act':''}" onclick="calView('list')">📋 List</button>
      <button class="vbtn ${calState.view==='month'?'act':''}" onclick="calView('month')">📅 Month</button>
    </div>
  </div>
  <div id="calBody">${calState.view==='list'?calList():calMonth()}</div>
  </div>`;
}

function calOpps(){return liveOpps().filter(o=>!calState.cat||o.cat===calState.cat).sort((a,b)=>new Date(a.close)-new Date(b.close));}

function calList(){
  const opps=calOpps();
  if(!opps.length) return'<p style="text-align:center;color:var(--mu);padding:60px">No upcoming deadlines.</p>';
  const byM={};
  opps.forEach(o=>{const k=o.close.slice(0,7);if(!byM[k])byM[k]=[];byM[k].push(o);});
  return Object.entries(byM).map(([ym,items])=>`
    <div style="margin-bottom:28px">
      <h3 class="lmh">${fmtMY(ym+'-01')}</h3>
      ${items.map(o=>{const u=urgInfo(o.close);const ci=catInfo(o.cat);return`
        <div class="li" onclick="openOpp('${o.id}')">
          <div class="li-dt">${fmtShort(o.close)}</div>
          <div><span class="li-ttl">${o.title}</span><span class="li-sub">${o.prov} · <span style="color:${ci.color}">${o.cat}</span></span></div>
          <div class="li-days" style="color:${u.c}">${u.label}</div>
        </div>`}).join('')}
    </div>`).join('');
}

function calMonth(){
  const {year,month,sel}=calState;
  const byD={};
  calOpps().forEach(o=>{if(!byD[o.close])byD[o.close]=[];byD[o.close].push(o);});
  const fd=new Date(year,month,1).getDay();
  const dim=new Date(year,month+1,0).getDate();
  const today=new Date().toISOString().split('T')[0];
  let cells='';
  for(let i=0;i<fd;i++) cells+=`<div class="cc"></div>`;
  for(let d=1;d<=dim;d++){
    const m0=String(month+1).padStart(2,'0'),d0=String(d).padStart(2,'0');
    const ds=`${year}-${m0}-${d0}`;
    const dO=byD[ds]||[];
    const u=dO.length?urgInfo(ds):null;
    cells+=`<div class="cc${dO.length?' has':''}${ds===today?' today':''}${ds===sel?' sel':''}"${dO.length?` onclick="calSel('${ds}')"`:''}><div class="cdn">${d}</div>${dO.length?`<span class="cdots" style="background:${u.bg};color:${u.c}">${dO.length} closing</span>`:''}</div>`;
  }
  const dp=sel&&byD[sel]?`<div class="day-panel"><h3>Closing on ${fmtD(sel)}</h3><div class="opp-grid" style="grid-template-columns:repeat(auto-fill,minmax(260px,1fr))">${byD[sel].map(o=>oppCard(o,true)).join('')}</div></div>`:'';
  return`<div class="cal-nav">
    <button class="cnav-btn" onclick="calPrev()">← Prev</button>
    <h2>${MONTHS[month]} ${year}</h2>
    <button class="cnav-btn" onclick="calNext()">Next →</button>
  </div>
  <div class="cgrid">${WDAYS.map(d=>`<div class="cdh">${d}</div>`).join('')}${cells}</div>${dp}`;
}

function calPrev(){if(calState.month===0){calState.month=11;calState.year--;}else calState.month--;calState.sel=null;document.getElementById('calBody').innerHTML=calState.view==='list'?calList():calMonth();}
function calNext(){if(calState.month===11){calState.month=0;calState.year++;}else calState.month++;calState.sel=null;document.getElementById('calBody').innerHTML=calState.view==='list'?calList():calMonth();}
function calView(v){calState.view=v;document.getElementById('calBody').innerHTML=v==='list'?calList():calMonth();document.querySelectorAll('.vbtn').forEach((b,i)=>b.classList.toggle('act',i===(v==='list'?0:1)));}
function calCat(v){calState.cat=v;document.getElementById('calBody').innerHTML=calState.view==='list'?calList():calMonth();}
function calSel(d){calState.sel=calState.sel===d?null:d;document.getElementById('calBody').innerHTML=calMonth();}

/* ═══════════════════════════════════════════════════════════
   TOOLS HUB
═══════════════════════════════════════════════════════════ */
function renderTools(){
  const tools=[
    {icon:"✅",title:"Eligibility Checker",desc:"Answer 5 questions and see every opportunity you're likely to qualify for.",p:"eligibility",c:"#166534",bg:"#dcfce7"},
    {icon:"🛡️",title:"Scam Awareness Checker",desc:"Describe an opportunity and get a risk assessment against known scam signals.",p:"scam",c:"#dc2626",bg:"#fee2e2"},
    {icon:"📋",title:"Checklist Generator",desc:"Get a personalised document checklist for learnerships, bursaries, or internships.",p:"checklist",c:"#1e40af",bg:"#dbeafe"},
    {icon:"🎯",title:"Bursary Probability Checker",desc:"Estimate your likelihood of qualifying for bursaries based on your marks and profile.",p:"bursary-checker",c:"#6b21a8",bg:"#f3e8ff"},
    {icon:"📅",title:"Deadline Calendar",desc:"View all upcoming closing dates in calendar or list view. Never miss a deadline.",p:"calendar",c:"#92400e",bg:"#fef3c7"},
    {icon:"🗺️",title:"Browse by Province",desc:"Filter opportunities by your province — see what's available near you.",p:"provinces",c:"#065f46",bg:"#ecfdf5"},
  ];
  return`<div class="page" style="max-width:940px;margin:0 auto;padding:40px 24px 80px">
  <h1 style="font-size:clamp(26px,5vw,42px);font-weight:900;color:var(--g);letter-spacing:-1px;margin-bottom:8px">Free Career Tools</h1>
  <p style="color:var(--mu);margin-bottom:36px;font-size:15px">Interactive tools to help you find, evaluate, and apply for opportunities — no login, no fees.</p>
  <div style="display:grid;grid-template-columns:repeat(auto-fill,minmax(380px,1fr));gap:18px">
    ${tools.map(t=>`<button onclick="go('${t.p}')" style="background:#fff;border:1px solid var(--bd);border-radius:14px;padding:26px;text-align:left;cursor:pointer;transition:all .2s;display:flex;flex-direction:column;gap:12px"
      onmouseover="this.style.borderColor='${t.c}';this.style.transform='translateY(-3px)';this.style.boxShadow='0 8px 24px ${t.c}22'"
      onmouseout="this.style.borderColor='var(--bd)';this.style.transform='none';this.style.boxShadow='none'">
      <div style="width:50px;height:50px;border-radius:12px;background:${t.bg};display:flex;align-items:center;justify-content:center;font-size:24px">${t.icon}</div>
      <div>
        <div style="font-family:'Fraunces',serif;font-size:19px;font-weight:700;color:var(--tx);margin-bottom:5px">${t.title}</div>
        <div style="font-size:13.5px;color:var(--mu);line-height:1.55">${t.desc}</div>
      </div>
      <span style="font-size:13px;color:${t.c};font-weight:700;margin-top:auto">Use tool →</span>
    </button>`).join('')}
  </div>
  </div>`;
}

/* ═══════════════════════════════════════════════════════════
   ELIGIBILITY CHECKER
═══════════════════════════════════════════════════════════ */
const QR={"grade-10":1,"grade-11":2,"matric":3,"diploma":4,"degree":5};
const OR={"Grade 11":2,"Matric":3,"Diploma":4,"Degree":5};

function renderElig(){
  return`<div class="page" style="max-width:700px;margin:0 auto;padding:40px 24px 80px">
  <button class="btn btn-ghost btn-sm" onclick="go('tools')" style="margin-bottom:16px">← Tools</button>
  <h1 style="font-size:clamp(22px,4vw,38px);font-weight:900;color:var(--g);letter-spacing:-1px;margin-bottom:8px">✅ Eligibility Checker</h1>
  <p style="color:var(--mu);margin-bottom:24px;font-size:15px">Answer 5 questions and we'll show you every opportunity you're likely to qualify for.</p>
  <div class="checker-card">
    ${[
      {id:"eq1",l:"Your highest qualification",opts:[["grade-10","Grade 10 or below"],["grade-11","Grade 11"],["matric","Matric / Grade 12"],["diploma","Diploma / N6"],["degree","Degree or higher"]]},
      {id:"eq2",l:"Your province",opts:[["","Any province"],...PROVS.filter(p=>p!=="Nationwide").map(p=>[p,p])]},
      {id:"eq3",l:"Employment status",opts:[["unemployed","Currently unemployed"],["student","Full-time student"],["employed","Currently employed"]]},
      {id:"eq4",l:"Opportunity type (optional)",opts:[["","Show everything"],...CATS.map(c=>[c.id,c.label])]},
    ].map(f=>`<div style="margin-bottom:18px">
      <label class="fl" for="${f.id}">${f.l}</label>
      <select class="flds" id="${f.id}">${f.opts.map(([v,l])=>`<option value="${v}">${l}</option>`).join('')}</select>
    </div>`).join('')}
    <button class="btn btn-primary btn-lg btn-full" onclick="runElig()">Find My Opportunities →</button>
  </div>
  <div id="eligRes"></div>
  </div>`;
}

function runElig(){
  const qual=document.getElementById('eq1')?.value||'matric';
  const prov=document.getElementById('eq2')?.value||'';
  const cat=document.getElementById('eq4')?.value||'';
  const uR=QR[qual]||3;
  const res=liveOpps().filter(o=>{
    if((OR[o.qual]||3)>uR) return false;
    if(prov&&o.prov!==prov&&o.prov!=='Nationwide') return false;
    if(cat&&o.cat!==cat) return false;
    return true;
  }).sort((a,b)=>new Date(a.close)-new Date(b.close));
  const er=document.getElementById('eligRes');
  if(!er)return;
  er.innerHTML=res.length?`<h2 style="font-family:'Fraunces',serif;font-size:22px;font-weight:700;color:var(--tx);margin:0 0 16px">${res.length} opportunities match your profile</h2>
  <div class="opp-grid">${res.map(o=>oppCard(o)).join('')}</div>`
  :`<div style="background:var(--sur);border:1px solid var(--bd);border-radius:var(--r);padding:28px;text-align:center">
    <p style="color:var(--mu);margin-bottom:14px;font-size:14px">No exact matches right now — try widening your filters.</p>
    <button class="btn btn-primary" onclick="go('guides')">Read Our SETA Guides →</button>
  </div>`;
  er.scrollIntoView({behavior:'smooth',block:'start'});
}

/* ═══════════════════════════════════════════════════════════
   SCAM CHECKER
═══════════════════════════════════════════════════════════ */
const SQ=[
  {id:"s1",q:"Were you asked to pay an application or registration fee?",yR:40,yF:"🚨 Application fee requested — no legitimate learnership ever charges fees."},
  {id:"s2",q:"Were personal documents requested before any interview?",yR:20,yF:"⚠️ Documents before interview — only provide after an official offer."},
  {id:"s3",q:"Was the only contact via WhatsApp from an unknown number?",yR:15,yF:"⚠️ WhatsApp-only contact — legitimate employers use official company email."},
  {id:"s4",q:"Was the contact email a Gmail, Yahoo, or Hotmail address?",yR:15,yF:"⚠️ Personal email used — real companies use @companyname.co.za"},
  {id:"s5",q:"Was the advertised stipend unusually high (R20 000+/month)?",yR:20,yF:"🚨 Unrealistic stipend — typical learnerships pay R3 500–R8 000/month."},
  {id:"s6",q:"Does the company have an official registered website?",nR:15,nF:"⚠️ No official website — verify the company at cipc.co.za first."},
  {id:"s7",q:"Were you pressured to apply within 24–48 hours?",yR:10,yF:"⚠️ Urgency pressure — a scammer tactic to prevent you from verifying."},
];
let sAns={};

function renderScam(){
  sAns={};
  return`<div class="page" style="max-width:680px;margin:0 auto;padding:40px 24px 80px">
  <button class="btn btn-ghost btn-sm" onclick="go('tools')" style="margin-bottom:16px">← Tools</button>
  <h1 style="font-size:clamp(22px,4vw,38px);font-weight:900;color:var(--g);letter-spacing:-1px;margin-bottom:8px">🛡️ Scam Awareness Checker</h1>
  <p style="color:var(--mu);margin-bottom:24px;font-size:15px">Describe an opportunity you saw advertised and we'll assess the risk level.</p>
  <div class="checker-card">
    ${SQ.map(q=>`<div style="margin-bottom:20px;padding-bottom:20px;border-bottom:1px solid #f3f4f6">
      <p style="font-size:14px;font-weight:600;color:var(--t2);margin-bottom:10px">${q.q}</p>
      <div class="rg">
        <button class="rb" id="s_${q.id}_y" onclick="sSet('${q.id}','y')">Yes</button>
        <button class="rb" id="s_${q.id}_n" onclick="sSet('${q.id}','n')">No</button>
      </div>
    </div>`).join('')}
    <button class="btn btn-primary btn-lg btn-full" onclick="runScam()">Check This Opportunity →</button>
  </div>
  <div id="scamRes"></div>
  </div>`;
}

function sSet(id,v){
  sAns[id]=v;
  ['y','n'].forEach(opt=>{const el=document.getElementById(`s_${id}_${opt}`);if(el)el.classList.toggle('sel',opt===v);});
}

function runScam(){
  let score=0;const flags=[];
  SQ.forEach(q=>{const a=sAns[q.id];if(q.yR&&a==='y'){score+=q.yR;flags.push(q.yF);}if(q.nR&&a==='n'){score+=q.nR;flags.push(q.nF);}});
  const level=score>=40?'HIGH':score>=20?'MEDIUM':'LOW';
  const cfg={HIGH:{cls:'high',tc:'#dc2626',advice:'Do NOT proceed. Multiple serious scam indicators. Report to SAPS immediately. Do not send money or documents.'},MEDIUM:{cls:'med',tc:'#d97706',advice:'Proceed with extreme caution. Verify the company independently at cipc.co.za before responding.'},LOW:{cls:'low',tc:'#166534',advice:'Relatively low risk. Still verify the company\'s official website before applying.'}};
  const s=cfg[level];
  const sr=document.getElementById('scamRes');
  if(!sr)return;
  sr.innerHTML=`<div class="risk-card ${s.cls}">
    <div style="font-family:'DM Mono',monospace;font-size:10.5px;text-transform:uppercase;letter-spacing:2px;color:${s.tc};margin-bottom:6px">Risk Assessment</div>
    <div class="risk-level" style="color:${s.tc}">${level} RISK</div>
    <p class="risk-advice">${s.advice}</p>
    ${flags.length?`<ul class="risk-flags">${flags.map(f=>`<li>${f}</li>`).join('')}</ul>`:''}
    <button class="btn btn-primary" onclick="go('directory')">Browse Verified Opportunities →</button>
  </div>`;
  sr.scrollIntoView({behavior:'smooth',block:'start'});
}

/* ═══════════════════════════════════════════════════════════
   CHECKLIST GENERATOR
═══════════════════════════════════════════════════════════ */
const CL={
  learnership:{label:"Learnership Application",steps:["Confirm eligibility — Matric, SA ID, unemployed, aged 18–35","Certify your ID at a police station (must be under 3 months old)","Certify your Matric certificate","Write your CV (max 2 pages, save as PDF)","Write your motivational letter (1 page, tailor to this specific company)","Get proof of residence (utility bill or bank statement, under 3 months)","Find and verify the official application link on the company website","Email subject: Application: [Learnership Name] — [Your Full Name]","Attach all documents — preferably as one PDF","Send before the closing date — late applications are not considered"]},
  bursary:{label:"Bursary Application",steps:["Confirm income eligibility — most require household income below R350 000/year","Certify your ID","Gather proof of income (parent/guardian payslips or affidavit if unemployed)","Collect your most recent academic results","Get proof of admission or registration from your institution","Write a specific motivational letter — name the bursary and field","Check the bursary terms — work-back clause, grade requirements, renewal conditions","Submit via the official application portal","Apply to at least 3–5 bursaries simultaneously to increase chances"]},
  internship:{label:"Internship Application",steps:["Confirm you hold the required qualification (diploma or degree)","Update your CV to include your qualification and any experience","Write a targeted cover letter for this specific company and role","Certify your qualification certificate","Get your official academic transcript from your institution","Prepare 2 references (lecturer, community leader, or previous employer)","Government internships: download and complete the Z83 form from dpsa.gov.za","Apply via the official company careers portal only"]},
  apprenticeship:{label:"Apprenticeship Application",steps:["Confirm you have Matric with Mathematics and Physical Science (min 50%)","Certify your ID and Matric certificate showing subject marks clearly","Obtain a medical fitness certificate from an occupational health practitioner","Write your CV highlighting any technical or practical experience","Write a targeted motivational letter — explain why you chose this trade","Apply via the official company or SETA website","Prepare for a trade aptitude test — some programmes include one at shortlisting"]}
};

function renderChecklist(){
  return`<div class="page" style="max-width:680px;margin:0 auto;padding:40px 24px 80px">
  <button class="btn btn-ghost btn-sm" onclick="go('tools')" style="margin-bottom:16px">← Tools</button>
  <h1 style="font-size:clamp(22px,4vw,38px);font-weight:900;color:var(--g);letter-spacing:-1px;margin-bottom:8px">📋 Application Checklist Generator</h1>
  <p style="color:var(--mu);margin-bottom:24px;font-size:15px">Select the opportunity type and get a personalised document and step checklist.</p>
  <div class="checker-card">
    <label class="fl">Opportunity type</label>
    <select class="flds" id="clType" onchange="showChecklist(this.value)" style="margin-bottom:20px">
      <option value="">Select type...</option>
      ${Object.entries(CL).map(([k,v])=>`<option value="${k}">${v.label}</option>`).join('')}
    </select>
  </div>
  <div id="clOutput"></div>
  </div>`;
}

function showChecklist(type){
  if(!type)return;
  const cl=CL[type];
  const output=document.getElementById('clOutput');
  if(!output)return;
  output.innerHTML=`<div class="cl-card">
    <h2 style="font-family:'Fraunces',serif;font-size:20px;font-weight:700;color:var(--tx);margin-bottom:16px">${cl.label} Checklist</h2>
    <ul class="cl-list">
      ${cl.steps.map((s,i)=>`<li class="cl-item" id="ci${i}">
        <input type="checkbox" id="cb${i}" onchange="toggleCl(${i})">
        <label for="cb${i}">${i+1}. ${s}</label>
      </li>`).join('')}
    </ul>
    <div style="display:flex;gap:12px;margin-top:20px;flex-wrap:wrap">
      <button class="btn btn-primary" onclick="window.print()">🖨️ Print Checklist</button>
      <button class="btn btn-ghost" onclick="resetCl()">Reset</button>
    </div>
    <p style="font-size:12px;color:var(--mu);margin-top:12px">Tip: Print this checklist and physically check off each item as you prepare.</p>
  </div>`;
}

function toggleCl(i){
  const li=document.getElementById(`ci${i}`);
  if(li)li.classList.toggle('done');
}

function resetCl(){
  document.querySelectorAll('.cl-item').forEach(el=>{
    el.classList.remove('done');
    el.querySelector('input').checked=false;
  });
}

/* ═══════════════════════════════════════════════════════════
   BURSARY PROBABILITY CHECKER
═══════════════════════════════════════════════════════════ */
function renderBursaryChecker(){
  return`<div class="page" style="max-width:680px;margin:0 auto;padding:40px 24px 80px">
  <button class="btn btn-ghost btn-sm" onclick="go('tools')" style="margin-bottom:16px">← Tools</button>
  <h1 style="font-size:clamp(22px,4vw,38px);font-weight:900;color:var(--g);letter-spacing:-1px;margin-bottom:8px">🎯 Bursary Probability Checker</h1>
  <p style="color:var(--mu);margin-bottom:24px;font-size:15px">Estimate your likelihood of qualifying for South African bursaries based on your academic profile and circumstances.</p>
  <div class="checker-card">
    ${[
      {id:"bp1",l:"Your Matric average percentage",opts:[["<50","Below 50%"],["50-59","50–59%"],["60-69","60–69%"],["70-79","70–79%"],["80+","80% or above"]]},
      {id:"bp2",l:"Mathematics performance",opts:[["none","Did not take Maths"],["<40","Below 40%"],["40-59","40–59%"],["60-69","60–69%"],["70+","70% or above"]]},
      {id:"bp3",l:"Household income range",opts:[["<50k","Below R50 000/year"],["50-150k","R50 000–R150 000/year"],["150-350k","R150 000–R350 000/year"],["350+","Above R350 000/year"]]},
      {id:"bp4",l:"Field of study interest",opts:[["engineering","Engineering / Sciences"],["it","Information Technology"],["health","Health Sciences"],["commerce","Commerce / Accounting"],["education","Education / Teaching"],["other","Other / Humanities"]]},
      {id:"bp5",l:"Current study status",opts:[["new","Entering university for first time"],["continuing","Currently studying"],["tvet","TVET college student"],["other","Other"]]},
    ].map(f=>`<div style="margin-bottom:18px">
      <label class="fl">${f.l}</label>
      <select class="flds" id="${f.id}">${f.opts.map(([v,l])=>`<option value="${v}">${l}</option>`).join('')}</select>
    </div>`).join('')}
    <button class="btn btn-primary btn-lg btn-full" onclick="runBursary()">Calculate My Probability →</button>
  </div>
  <div id="bursaryRes"></div>
  </div>`;
}

function runBursary(){
  const avg=document.getElementById('bp1')?.value;
  const maths=document.getElementById('bp2')?.value;
  const income=document.getElementById('bp3')?.value;
  const field=document.getElementById('bp4')?.value;
  const status=document.getElementById('bp5')?.value;

  let score=0;
  // Academic score
  if(avg==='80+') score+=30; else if(avg==='70-79') score+=25; else if(avg==='60-69') score+=18; else if(avg==='50-59') score+=10; else score+=3;
  // Maths
  if(maths==='70+') score+=20; else if(maths==='60-69') score+=15; else if(maths==='40-59') score+=8; else if(maths==='<40') score+=3;
  // Income
  if(income==='<50k') score+=25; else if(income==='50-150k') score+=20; else if(income==='150-350k') score+=12; else score+=0;
  // Field — high demand boosts
  if(['engineering','it','health'].includes(field)) score+=15; else if(field==='commerce') score+=10; else score+=5;
  // Status
  if(status==='new') score+=10; else if(status==='tvet') score+=8; else if(status==='continuing') score+=6;

  const maxScore=100;
  const pct=Math.min(score,maxScore);
  const level=pct>=70?'High':pct>=45?'Medium':'Low';
  const colors={High:'#166534',Medium:'#d97706',Low:'#dc2626'};
  const bgs={High:'#dcfce7',Medium:'#fef3c7',Low:'#fee2e2'};
  const advice={
    High:"Your profile is strong for bursary applications. Focus on applying to merit-based bursaries in your field. Prioritise companies in engineering, banking, and mining — they offer the most bursaries.",
    Medium:"You have a reasonable chance, especially for need-based bursaries. NSFAS is a strong option if your household income qualifies. Apply to 5–10 bursaries to maximise your chances.",
    Low:"Your current profile may need strengthening. Consider improving your Matric results (supplementary exams), applying for NSFAS if eligible, or starting at a TVET college where funding is more accessible."
  };

  const bursaryList=liveOpps().filter(o=>o.cat==='bursary').slice(0,3);
  const br=document.getElementById('bursaryRes');
  if(!br)return;
  br.innerHTML=`<div style="background:${bgs[level]};border:1px solid ${colors[level]}44;border-radius:var(--r);padding:24px;margin-bottom:20px">
    <div style="font-family:'DM Mono',monospace;font-size:10.5px;text-transform:uppercase;letter-spacing:2px;color:${colors[level]};margin-bottom:6px">Your Bursary Probability</div>
    <div style="font-family:'Fraunces',serif;font-size:32px;font-weight:900;color:${colors[level]};margin-bottom:4px">${pct}% — ${level} Match</div>
    <div class="prob-bar-wrap"><div class="prob-bar" style="width:${pct}%;background:${colors[level]}"></div></div>
    <p style="font-size:14px;color:var(--t2);line-height:1.7;margin-bottom:14px">${advice[level]}</p>
    ${bursaryList.length?`<h3 style="font-family:'Fraunces',serif;font-size:16px;font-weight:700;margin-bottom:12px">Current Open Bursaries</h3>
    <div style="display:flex;flex-direction:column;gap:10px">${bursaryList.map(o=>`
      <div onclick="openOpp('${o.id}')" style="background:#fff;border:1px solid var(--bd);border-radius:8px;padding:12px 14px;cursor:pointer">
        <div style="font-weight:600;font-size:14px;margin-bottom:2px">${o.title}</div>
        <div style="font-size:12px;color:var(--mu)">${o.prov} · ${o.stipend}</div>
      </div>`).join('')}</div>`:''}
  </div>
  <button class="btn btn-primary" onclick="go('directory');setTimeout(()=>{document.getElementById('dc').value='bursary';applyDir()},60)">Browse All Bursaries →</button>`;
  br.scrollIntoView({behavior:'smooth',block:'start'});
}

/* ═══════════════════════════════════════════════════════════
   GUIDES
═══════════════════════════════════════════════════════════ */
function renderGuides(){
  return`<div class="page" style="max-width:1000px;margin:0 auto;padding:40px 24px 80px">
  <h1 style="font-size:clamp(26px,5vw,42px);font-weight:900;color:var(--g);letter-spacing:-1px;margin-bottom:8px">SETA Guides & Career Resources</h1>
  <p style="color:var(--mu);margin-bottom:32px;font-size:15px">Plain-language guides explaining South Africa's career systems — written for job seekers, not HR departments.</p>
  <div class="guide-grid">
    ${GUIDES.map(g=>`<div class="guide-card" onclick="openGuide('${g.id}')">
      <div class="gc-meta">📖 ${g.cat.replace('-',' ')} · ${g.read} read</div>
      <h3 class="gc-title">${g.title}</h3>
      <p class="gc-intro">${g.intro}</p>
      <span class="gc-lnk">Read guide →</span>
    </div>`).join('')}
  </div>
  </div>`;
}

function renderGuideDetail(){
  const g=_guide;
  if(!g)return'';
  const related=liveOpps().slice(0,3);
  return`<div class="page" style="max-width:var(--cw);margin:0 auto;padding:40px 24px 80px">
  <div class="breadcrumb">
    <button onclick="go('home')">Home</button><span class="sep">›</span>
    <button onclick="go('guides')">Guides</button><span class="sep">›</span>
    <span style="color:var(--t2)">${g.title}</span>
  </div>
  <div class="gc-meta" style="font-size:11px;margin-bottom:12px">📖 ${g.cat.replace('-',' ')} · ${g.read} read</div>
  <h1 style="font-family:'Fraunces',serif;font-size:clamp(24px,4vw,38px);font-weight:900;color:var(--tx);line-height:1.2;letter-spacing:-1.5px;margin-bottom:22px">${g.title}</h1>
  <div style="background:#f0fdf4;border:1px solid #bbf7d0;border-radius:var(--r);padding:18px 20px;margin-bottom:26px">
    <p style="font-size:15px;color:var(--gmid);font-weight:500;line-height:1.7;margin:0">${g.intro}</p>
  </div>
  <style>.gbd h2{font-family:'Fraunces',serif;font-size:19px;font-weight:700;color:var(--tx);margin:24px 0 12px}.gbd p{font-size:15px;color:var(--t2);line-height:1.85;margin-bottom:14px}.gbd ul{padding-left:22px;margin-bottom:14px}.gbd li{margin-bottom:6px;line-height:1.6;color:var(--t2);font-size:15px}.gbd strong{color:var(--tx)}</style>
  <div class="gbd">${g.body}</div>
  <div style="background:var(--g);border-radius:var(--r);padding:24px;margin:32px 0">
    <h3 style="font-family:'Fraunces',serif;font-size:18px;font-weight:700;color:#fff;margin-bottom:8px">Ready to Apply?</h3>
    <p style="color:#86efac;font-size:14px;margin-bottom:14px">Browse all current verified opportunities in South Africa.</p>
    <button class="btn btn-amber" onclick="go('directory')">Browse All Opportunities →</button>
  </div>
  <h2 style="font-family:'Fraunces',serif;font-size:22px;font-weight:700;color:var(--g);margin-bottom:14px">Current Open Opportunities</h2>
  <div style="display:flex;flex-direction:column;gap:10px">
    ${related.map(o=>`<div onclick="openOpp('${o.id}')" style="background:#fff;border:1px solid var(--bd);border-radius:9px;padding:14px 16px;cursor:pointer;display:flex;justify-content:space-between;align-items:center;transition:all .15s" onmouseover="this.style.borderColor='var(--g)'" onmouseout="this.style.borderColor='var(--bd)'">
      <div><div style="font-weight:600;font-size:14px;color:var(--tx);margin-bottom:2px">${o.title}</div><div style="font-size:12px;color:var(--mu)">${o.prov} · ${o.stipend}</div></div>
      <span style="font-size:13px;color:var(--g);font-weight:600;flex-shrink:0">View →</span>
    </div>`).join('')}
  </div>
  </div>`;
}

/* ═══════════════════════════════════════════════════════════
   PROVINCES
═══════════════════════════════════════════════════════════ */
function renderProvinces(){
  const live=liveOpps();
  const provData=PROVS.filter(p=>p!=='Nationwide').map(p=>{
    const cnt=live.filter(o=>o.prov===p||o.prov==='Nationwide').length;
    return{p,cnt};
  });
  return`<div class="page" style="max-width:var(--mw);margin:0 auto;padding:40px 24px 80px">
  <h1 style="font-size:clamp(26px,5vw,42px);font-weight:900;color:var(--g);letter-spacing:-1px;margin-bottom:8px">Browse by Province</h1>
  <p style="color:var(--mu);margin-bottom:32px;font-size:15px">Select your province to see all relevant opportunities</p>
  <div class="prov-grid">
    ${provData.map(({p,cnt})=>`<button class="prov-tile" onclick="filterByProvince('${p}')">
      <h3>📍 ${p}</h3>
      <p>${cnt} opportunities available</p>
    </button>`).join('')}
  </div>
  <div id="provResults"></div>
  </div>`;
}

function filterByProvince(prov){
  const res=liveOpps().filter(o=>o.prov===prov||o.prov==='Nationwide').sort((a,b)=>new Date(a.close)-new Date(b.close));
  const pr=document.getElementById('provResults');
  if(!pr)return;
  pr.innerHTML=`<h2 style="font-family:'Fraunces',serif;font-size:24px;font-weight:700;color:var(--g);margin-bottom:20px;border-top:2px solid var(--glt);padding-top:24px">Opportunities in ${prov} (${res.length} found)</h2>
  <div class="opp-grid">${res.map(o=>oppCard(o)).join('')}</div>`;
  pr.scrollIntoView({behavior:'smooth',block:'start'});
}

/* ═══════════════════════════════════════════════════════════
   SAVED
═══════════════════════════════════════════════════════════ */
function renderSaved(){
  const ids=getSaved();
  const saved=OPPS.filter(o=>ids.includes(o.id));
  if(!saved.length) return`<div class="page" style="max-width:var(--mw);margin:0 auto;padding:40px 24px 80px">
    <h1 style="font-size:clamp(26px,5vw,40px);font-weight:900;color:var(--g);letter-spacing:-1px;margin-bottom:8px">🔖 Saved Opportunities</h1>
    <div class="saved-empty">
      <div class="se-ico">🔖</div>
      <h3 style="font-family:'Fraunces',serif;font-size:24px;font-weight:700;color:var(--t2);margin-bottom:10px">No saved opportunities yet</h3>
      <p style="font-size:14px;margin-bottom:24px">Click the 🔲 button on any opportunity card to save it for later.</p>
      <button class="btn btn-primary" onclick="go('directory')">Browse Opportunities</button>
    </div>
  </div>`;
  return`<div class="page" style="max-width:var(--mw);margin:0 auto;padding:40px 24px 80px">
  <h1 style="font-size:clamp(26px,5vw,40px);font-weight:900;color:var(--g);letter-spacing:-1px;margin-bottom:8px">🔖 Saved Opportunities</h1>
  <p style="color:var(--mu);margin-bottom:24px">${saved.length} saved opportunity${saved.length!==1?'s':''}</p>
  <div class="opp-grid">${saved.map(o=>oppCard(o)).join('')}</div>
  </div>`;
}

/* ═══════════════════════════════════════════════════════════
   ABOUT
═══════════════════════════════════════════════════════════ */
function renderAbout(){
  return`<div class="page" style="max-width:var(--cw);margin:0 auto;padding:40px 24px 80px">
  <h1 style="font-family:'Fraunces',serif;font-size:clamp(28px,5vw,44px);font-weight:900;color:var(--g);letter-spacing:-1.5px;margin-bottom:8px">About OpportunitiesZA</h1>
  <div class="about-accent"></div>
  <p style="font-size:16px;color:var(--t2);line-height:1.85;margin-bottom:18px"><strong>OpportunitiesZA</strong> exists to make South Africa's career opportunity landscape simple and accessible. The biggest barrier between an unemployed young South African and their first qualification is not effort — it's information.</p>
  <p style="font-size:15px;color:#4b5563;line-height:1.85;margin-bottom:32px">We publish verified listings, explain complex SETA systems in plain language, and build free tools so users can figure out what they qualify for — without paying anyone or sending their documents anywhere.</p>
  <h2 style="font-family:'Fraunces',serif;font-size:22px;font-weight:700;color:var(--tx);margin-bottom:16px">What We Do</h2>
  <ul class="acl">
    ${["Publish verified opportunity listings sourced from official company and government websites","Explain the SETA system, NQF levels, and career pathways in plain South African English","Maintain a deadline calendar so users never miss a closing date","Provide 6 free tools including eligibility checker, scam checker, and checklist generator","Verify every listing before publishing and archive expired listings promptly"].map(i=>`<li>${i}</li>`).join('')}
  </ul>
  <div class="disc-box">
    <h3>⚠️ Disclaimer</h3>
    <p>OpportunitiesZA is an independent information platform. We are not an employer, recruiter, recruitment agency, or government department. We do not collect personal documents, charge fees of any kind, or guarantee placement in any programme. Always verify opportunity details directly with the official source before applying or submitting personal information.</p>
  </div>
  <h2 style="font-family:'Fraunces',serif;font-size:22px;font-weight:700;color:var(--tx);margin-bottom:14px">Contact</h2>
  <p style="font-size:14.5px;color:var(--t2);line-height:1.8">
    General enquiries: <a href="mailto:contact@opportunitiesza.co.za" style="color:var(--g);font-weight:600">contact@opportunitiesza.co.za</a><br>
    Report a scam, submit an opportunity, or enquire about partnerships via the same email. We respond within 48 hours.
  </p>
  </div>`;
}

/* ═══════════════════════════════════════════════════════════
   BOOT
═══════════════════════════════════════════════════════════ */
renderLoading();
loadContent()
  .then(()=>AppRouter.start())
  .catch(err=>{
    console.error('Content load failed',err);
    renderLoadError(err);
  });
</script>
</body>
</html>
````

## File: package.json
````json
{
  "name": "opportunitiesza",
  "version": "1.0.0",
  "private": true,
  "description": "South Africa career opportunity intelligence platform for learnerships, bursaries, internships, and apprenticeships.",
  "scripts": {
    "validate": "node scripts/validate-content.js",
    "sitemap": "node sitemap-generator.js",
    "check": "npm run validate && npm run links && npm run sitemap",
    "links": "node scripts/check-links.js",
    "source:audit": "node scripts/source-audit.js",
    "source:health": "node scripts/source-health-check.js",
    "crawl": "node crawler/index.js"
  },
  "engines": {
    "node": ">=18"
  },
  "dependencies": {
    "axios": "1.18.0",
    "cheerio": "1.2.0",
    "pdf-parse": "1.1.4",
    "p-limit": "6.2.0",
    "uuid": "11.1.1"
  }
}
````

## File: README.md
````markdown
# 🧭 OpportunitiesZA — Complete Deployment Guide

South Africa's career opportunity intelligence platform for learnerships, bursaries, internships, and apprenticeships.

**Live site:** https://opportunitiesza.co.za  
**Repository:** https://github.com/Ituthema/seta-careers-platform1  
**Stack:** Vanilla HTML / CSS / JS · GitHub Pages · Zero build step  

---

## 📁 Project Structure

```
/opportunitiesza/
│
├── index.html              ← Main SPA (the entire website)
├── 404.html                ← GitHub Pages SPA routing fallback
├── robots.txt              ← SEO crawl directives
├── sitemap.xml             ← Auto-generated (run sitemap-generator.js)
├── sitemap-generator.js    ← Node script to regenerate sitemap
├── validate-content.js      ← Checks data, index loader, and sitemap slugs
├── CNAME                   ← Custom domain (create after buying domain)
├── README.md               ← This file
│
├── /data/
│   ├── opportunities.json  ← All opportunity records
│   ├── guides.json         ← All guide articles
│   ├── categories.json     ← Category metadata
│   └── provinces.json      ← Province list
│
└── /assets/
    ├── /images/            ← OG images, icons
    └── og-default.jpg      ← 1200×630 social share image
```

---

## 🚀 Initial Deployment (GitHub Pages)

### Prerequisites (Termux)
```bash
# Install git if not already installed
pkg install git

# Install Node.js for the sitemap generator
pkg install nodejs

# Configure git identity
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# Fix Termux safe directory warning
git config --global --add safe.directory /storage/emulated/0/Download/opportunitiesza
```

### First Push
```bash
# Navigate to project folder
cd /storage/emulated/0/Download/opportunitiesza

# Initialise and push
rm -rf .git
git init
git add .
git commit -m "Initial deployment — OpportunitiesZA"
git branch -M main
git remote add origin https://github.com/Ituthema/seta-careers-platform1.git
git push -u origin main

# When prompted:
#   Username: ituthema
#   Password: YOUR_GITHUB_PERSONAL_ACCESS_TOKEN
#   (Not your account password — generate at github.com/settings/tokens)
```

### Enable GitHub Pages
1. Go to your repository on GitHub
2. **Settings → Pages**
3. Source: **Deploy from a branch**
4. Branch: **main**, Folder: **/ (root)**
5. Click **Save**
6. Site will be live at: `https://ituthema.github.io/seta-careers-platform1/`

---

## 🔑 Set Up SSH (Recommended — No Token Prompts)

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your@email.com"
# Press Enter for all prompts (no passphrase needed)

# View and copy your public key
cat ~/.ssh/id_ed25519.pub

# Add to GitHub:
# github.com → Settings → SSH and GPG Keys → New SSH Key
# Paste the public key, save

# Switch your remote to SSH
git remote set-url origin git@github.com:Ituthema/seta-careers-platform1.git

# Test connection
ssh -T git@github.com
# Should see: "Hi Ituthema! You've successfully authenticated."

# From now on, push with no prompts:
git push
```

---

## 🌐 Custom Domain Setup

### Step 1 — Buy Your Domain
Recommended SA registrars (cheapest to most features):
- **Hetzner.co.za** — ~R99/year for .co.za
- **Afrihost.com** — ~R150/year, good DNS management
- **Domains.co.za** — ~R129/year

Recommended names:
- `opportunitiesza.co.za`
- `setacareershub.co.za`
- `careerssouthafrica.co.za`

### Step 2 — Create CNAME File
```bash
# In your project root
echo "opportunitiesza.co.za" > CNAME
git add CNAME
git commit -m "Add custom domain CNAME"
git push
```

### Step 3 — Set DNS Records at Your Registrar
In your registrar's DNS management panel, add these records:

| Type  | Host | Value                |
|-------|------|----------------------|
| A     | @    | 185.199.108.153      |
| A     | @    | 185.199.109.153      |
| A     | @    | 185.199.110.153      |
| A     | @    | 185.199.111.153      |
| CNAME | www  | ituthema.github.io   |

### Step 4 — Configure in GitHub Pages
1. Settings → Pages → Custom domain
2. Enter: `opportunitiesza.co.za`
3. Click Save, wait for DNS check ✓
4. **Enable "Enforce HTTPS"** once verified

DNS propagation takes 15 minutes to 48 hours.

---

## 📝 Daily Content Workflow

### Add New Opportunity (Termux)
```bash
cd /storage/emulated/0/Download/opportunitiesza

# Edit the opportunities data file
nano data/opportunities.json

# Add new record at the TOP of the array using this template:
# {
#   "id": "023",
#   "slug": "company-type-2026",
#   "title": "Company Name Learnership 2026",
#   "category": "learnership",
#   "sector": "MICT SETA",
#   "province": "Gauteng",
#   "qualification_level": "Matric",
#   "stipend": "R4 000/month",
#   "closing_date": "2026-09-30",
#   "application_url": "https://company.co.za/careers",
#   "source_name": "Company Official Website",
#   "source_url": "https://company.co.za/careers",
#   "verified": true,
#   "featured": false,
#   "expired": false,
#   "description": "...",
#   "eligibility": ["..."],
#   "documents_needed": ["..."],
#   "tags": ["matric", "gauteng"],
#   "posted_date": "2026-05-01",
#   "updated_date": "2026-05-01"
# }

# Validate JSON before pushing
python3 -m json.tool data/opportunities.json > /dev/null && echo "✅ Valid JSON" || echo "❌ JSON Error — fix before pushing"

# Validate that index.html loads /data/ and live opportunity slugs match sitemap.xml
node validate-content.js

# Push live
git add data/opportunities.json
git commit -m "Add [Company] [type] - closing [date]"
git push
```

### Regenerate Sitemap After Publishing
`data/` is the canonical content source. After editing opportunities or guides, regenerate the sitemap from the JSON files and run the slug agreement check before publishing.

```bash
node sitemap-generator.js
node validate-content.js
git add sitemap.xml
git commit -m "Update sitemap"
git push
```

### Scaled Quality Workflow
```bash
# Validate content records, dates, slugs, URLs, and required fields
npm run validate

# Check official source and application URLs
npm run links

# Regenerate sitemap.xml from active opportunities and guides
npm run sitemap

# Run the full release check before opening a pull request
npm run check
```

For the full operating cadence, contributor checklist, and scale-up backlog, see [`docs/UPSCALE_WORKFLOW.md`](docs/UPSCALE_WORKFLOW.md).

### Nano Quick Reference (Termux Editor)
| Keys | Action |
|------|--------|
| `CTRL + O` | Save file |
| `CTRL + X` | Exit |
| `CTRL + K` | Cut line |
| `CTRL + U` | Paste line |
| `CTRL + W` | Search |
| `ALT + /`  | Jump to end |

### One-Command Publish Alias
Add to `~/.bashrc` for a single-command publish:
```bash
echo "alias publish='cd /storage/emulated/0/Download/opportunitiesza && git add . && git commit -m \"Content update \$(date +%Y-%m-%d)\" && git push'" >> ~/.bashrc
source ~/.bashrc

# Then use:
publish
```

---

## 🔍 Google Search Console Setup

### Add Property
1. Go to [search.google.com/search-console](https://search.google.com/search-console)
2. Add property → **URL prefix**
3. Enter: `https://opportunitiesza.co.za` (your custom domain)

### Verify Ownership
```bash
# Download the HTML verification file from GSC
# Place it in your project root
cp ~/Downloads/google[XXXXX].html /storage/emulated/0/Download/opportunitiesza/
git add google[XXXXX].html
git commit -m "Add GSC verification"
git push
# Then click Verify in GSC
```

### Submit Sitemap
1. GSC → Sitemaps → Add a new sitemap
2. Enter: `sitemap.xml`
3. Click Submit

### Request Indexing for New Pages
After publishing important new pages:
1. GSC → URL Inspection
2. Paste the URL
3. Click **Request Indexing**

Do this for: homepage, all category pages, and first 10 opportunity pages in week 1.

---

## 📊 Google Analytics 4 Setup

1. Create a GA4 property at [analytics.google.com](https://analytics.google.com)
2. Get your Measurement ID (format: `G-XXXXXXXXXX`)
3. Add to `index.html` inside `<head>`:

```html
<!-- Google Analytics 4 -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){ dataLayer.push(arguments); }
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```

---

## 💰 Google AdSense Setup

### Requirements Before Applying
- Minimum 15–20 original published articles/guides
- Working About page and Privacy Policy page
- Site live for at least 2–4 weeks
- No copied content — all original

### Apply
1. Go to [adsense.google.com](https://adsense.google.com)
2. Add your site URL
3. Add the AdSense script to `<head>` of `index.html`:
```html
<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-XXXXXXXXXX" crossorigin="anonymous"></script>
```
4. Wait for approval (1–7 days typically)

---

## 🗑️ Mark Expired Opportunities

Opportunities past their closing date should be marked expired (not deleted — they retain SEO value):

```bash
nano data/opportunities.json
# Find the expired opportunity and change:
#   "expired": false  →  "expired": true
# Save and push
git add data/opportunities.json
git commit -m "Archive expired: [opportunity name]"
git push
```

The website automatically filters out expired opportunities from all listings, but the URL remains live showing an "expired" notice.

---

## 📱 WhatsApp Broadcast Setup

### Phase 1 — Personal Broadcast List
1. Create a WhatsApp Broadcast List
2. Add contacts who want daily opportunity updates
3. Use the daily template from the documentation
4. Max 256 contacts per broadcast list

### Phase 2 — WhatsApp Business
1. Download WhatsApp Business app
2. Register with a dedicated phone number
3. Set up: Business name, description, website URL, hours
4. Enables catalogue and quick-reply features

### Phase 3 — WhatsApp Channel (Unlimited)
1. In WhatsApp: Updates tab → + → New Channel
2. Name it "OpportunitiesZA Daily Updates"
3. Add channel link to your website and social bios
4. No follower limit, no phone number needed from followers

---

## 📈 Monthly Checklist

### Content
- [ ] Add 5–10 new opportunity records per week
- [ ] Publish 2–3 new guide articles per week
- [ ] Mark expired opportunities (check weekly)
- [ ] Regenerate sitemap.xml after publishing

### SEO
- [ ] Check GSC for new keyword impressions
- [ ] Request indexing for new pages
- [ ] Review pages with high impressions but low CTR — rewrite meta descriptions
- [ ] Check for crawl errors in GSC Coverage report

### Distribution
- [ ] Post 1–2 TikTok/Reels videos daily
- [ ] Send 1 WhatsApp broadcast daily
- [ ] Post in 3–5 Facebook groups weekly
- [ ] Send weekly email digest (Monday)

### Technical
- [ ] Validate opportunities.json after every edit
- [ ] Run `npm run links`, `node sitemap-generator.js`, and `node validate-content.js` before publishing content changes
- [ ] Test site on mobile after any HTML changes
- [ ] Check that apply links are not broken
- [ ] Review Core Web Vitals in GSC

---

## 🐛 Troubleshooting

### Site not updating after `git push`
GitHub Pages can take 1–5 minutes. Check status at:  
`https://github.com/Ituthema/seta-careers-platform/actions`

### JSON validation error
```bash
python3 -m json.tool data/opportunities.json
# Shows exactly which line has the error
```

### Sitemap or slug validation error
```bash
node sitemap-generator.js
node validate-content.js
# Confirms live opportunity slugs in data/opportunities.json match sitemap.xml and that index.html loads /data/opportunities.json
```

Common mistakes:
- Missing comma after a field
- Trailing comma after the last item in an array
- Unescaped quotes inside strings — use `\"` instead of `"`

### Custom domain not working
1. Check DNS records are saved correctly at your registrar
2. Run: `nslookup opportunitiesza.co.za` — should show GitHub IPs
3. Wait up to 48 hours for DNS propagation
4. Ensure CNAME file exists in repo root with just your domain

### Mobile menu not closing
Hard refresh: `CTRL+SHIFT+R` in browser, or clear site data.

---

## 📞 Contacts & Resources

| Resource | URL |
|----------|-----|
| GitHub Pages docs | https://docs.github.com/en/pages |
| Google Search Console | https://search.google.com/search-console |
| Google Analytics | https://analytics.google.com |
| Google AdSense | https://adsense.google.com |
| PageSpeed Insights | https://pagespeed.web.dev |
| Rich Results Test | https://search.google.com/test/rich-results |
| CIPC (verify companies) | https://www.cipc.co.za |
| DPSA vacancies | https://www.dpsa.gov.za/dpsa2g/vacancies.asp |
| NSFAS applications | https://mynsfas.nsfas.org.za |
| myNSFAS portal | https://mynsfas.nsfas.org.za |

---

## 📄 Licence

This project is for the exclusive use of the repository owner.  
Content is original and verified against official South African government and corporate sources.  
All opportunity listings link to official sources — we are an information platform, not an employer.

---

*OpportunitiesZA · South Africa's Career Opportunity Intelligence Platform · 2026*
# seta-careers-platform1
````

## File: Repository Workflow Optimization Framework.txt
````
Repository Workflow Optimization Framework
Codex Context / Token Usage Reduction Framework
Based on the conversations in this project, there are two major themes that were developed in depth:

1. Repository Workflow Optimization Framework


2. Codex Context / Token Usage Reduction Framework



These discussions evolved into a methodology for building large repositories with AI while minimizing unnecessary context loading, reducing prompt sizes, increasing task completion rates, and making future AI sessions significantly cheaper and more accurate.


---

PART I — Repository Workflow Optimization Framework

The repository workflow was designed around a modular AI-first development process.

Instead of treating the repository as one large project, it is divided into independent systems.

The goal is:

minimize AI context

isolate failures

improve maintainability

allow parallel development

reduce hallucinations

improve debugging

improve testing



---

1 Repository as Independent Systems

Rather than:

Repository
    ↓
Everything connected

the repository becomes

Repository

Frontend
Backend
Data Sources
Scrapers
API
Validation
Scheduler
Parser
Documentation
Tests
Deployment

Each system owns its own:

documentation

architecture

tests

configuration

AI context


Meaning Codex only loads the section being modified.


---

2 Layered Architecture

Development is separated into layers.

Example

UI

↓

Business Logic

↓

Services

↓

Data Layer

↓

Storage

↓

Configuration

Each layer communicates through interfaces rather than directly.

Benefits

fewer dependencies

easier debugging

smaller AI context

isolated testing



---

3 Feature Isolation

Every feature is treated as its own module.

Instead of

Everything imports everything

Use

Feature A

docs
tests
logic
config

Feature B

docs
tests
logic
config

AI only needs Feature B to modify Feature B.


---

4 Task-Based Development

Large prompts become hundreds of small tasks.

Example

Instead of

Build scraping system

Split into

Task 1

Create source loader

Task 2

Validate sources

Task 3

Create parser interface

Task 4

Implement PDF parser

Task 5

Unit tests

Each task loads dramatically fewer files.


---

5 Strict Repository Boundaries

Every task clearly defines

Allowed folders

Forbidden folders

Files to modify

Files to create

Expected outputs

Success criteria

Rollback strategy

This prevents Codex from exploring the entire repository.


---

6 Documentation-Driven Development

Documentation becomes the primary source of truth.

Examples

docs/

architecture

modules

rules

context

workflow

templates

contracts

interfaces

Instead of loading 300 source files, AI loads a few documentation files.


---

7 Phase-Based Development

Development progresses in phases.

Example

Phase 0

Repository structure

Phase 1

Infrastructure

Phase 2

Core engine

Phase 3

Scrapers

Phase 4

API

Phase 5

UI

Phase 6

Testing

Phase 7

Deployment

Each phase has independent documentation.


---

PART II — Codex Token Usage Reduction Framework

This became one of the major focuses of the project.

Goal:

Reduce context size while increasing output quality.


---

Core Principle

Never make Codex understand the whole repository.

Instead

Make Codex understand one task.


---

1 Context Isolation

Every task receives only

Objective

Files

Dependencies

Expected outputs

Success criteria

Nothing else.

Instead of

Understand entire repository.

Provide

Modify:

scripts/parser.js

Only.

Huge token reduction.


---

2 Context Hierarchy

The framework defines multiple context levels.

Repository Context

↓

Module Context

↓

Feature Context

↓

Task Context

↓

File Context

↓

Function Context

Codex should only receive the lowest necessary level.


---

3 Context Files

Dedicated documentation files were proposed.

Examples

docs/context/

repository.md

frontend.md

backend.md

parser.md

api.md

scraper.md

scheduler.md

validation.md

Codex reads only one context file.


---

4 AI Rules

Create

docs/rules/

Containing

Naming conventions

Folder rules

Coding standards

Architecture rules

Import rules

Testing rules

Documentation rules

Instead of explaining rules every prompt.


---

5 Prompt Templates

Instead of rewriting prompts.

Store templates.

Example

docs/templates/

bug-fix.md

feature.md

refactor.md

documentation.md

testing.md

review.md

optimization.md


---

6 Scope Limitation

Every task contains

Scope

Out of scope

Forbidden actions

Example

ONLY modify

scripts/parser.js

Do NOT modify

data

UI

tests

package.json

Codex explores dramatically fewer files.


---

7 Explicit Constraints

Tasks define

May create

May edit

Must not edit

Allowed dependencies

Forbidden dependencies

Expected filenames

Expected functions

Maximum files modified

This prevents unnecessary exploration.


---

8 Small Independent Tasks

Large

Implement scraper

becomes

Task A

Create interface

Task B

Validate URLs

Task C

Normalize output

Task D

Error handling

Task E

Tests

Task F

Documentation

Each prompt becomes tiny.


---

9 Task Chaining

Rather than

Huge prompt

Use

Task 1

↓

Task 2

↓

Task 3

↓

Task 4

Each task references outputs from previous tasks instead of repeating full context.


---

10 Repository Memory

Persistent documentation replaces repeated explanations.

Examples

architecture.md

coding-rules.md

interfaces.md

contracts.md

Codex loads these instead of rediscovering the repository.


---

11 Interface Contracts

Modules communicate through documented interfaces.

Example

Input

↓

Validation

↓

Parser

↓

Normalizer

↓

Storage

Each interface is documented.

Codex doesn't inspect every module.


---

12 Stable APIs

Document

Inputs

Outputs

Types

Errors

Examples

No source-code exploration required.


---

13 AI Context Compression

Instead of

100 files

Summarize into

One architecture file

One interface file

One workflow file

One module file

This dramatically reduces prompt size.


---

14 Documentation Before Code

Before implementation create

Architecture

Contracts

Interfaces

Workflow

Dependencies

Then implementation.

AI reads documentation instead of discovering code.


---

15 Standard Task Structure

Every Codex prompt follows a fixed structure.

1. Objective


2. Scope


3. Files allowed


4. Files forbidden


5. Expected output


6. Constraints


7. Acceptance criteria



This standardization reduces ambiguity and unnecessary context.


---

16 Dependency Mapping

Document module dependencies explicitly.

Instead of allowing AI to infer relationships, provide a dependency graph showing which modules interact and which are isolated. This reduces exploratory code reading.


---

17 Incremental Repository Indexing

Maintain lightweight index files that summarize repository structure, major modules, and entry points. AI consults these indexes first before any code, avoiding repeated directory traversal.


---

18 Architecture Decision Records (ADRs)

Capture important architectural decisions in dedicated documents. Future tasks reference ADRs instead of rediscovering why design choices were made, preventing repeated analysis.


---

19 Success Criteria-Driven Prompts

Every task specifies measurable completion criteria such as:

Required files created or modified.

Tests passing.

Documentation updated.

No unrelated files changed.


This limits over-implementation.


---

20 Failure and Rollback Guidance

Include expected failure handling and rollback instructions within task prompts so Codex can avoid cascading changes when a step cannot be completed.


---

21 Documentation Directory Structure

A dedicated documentation hierarchy was proposed to support context compression:

docs/
├── architecture/
├── context/
├── rules/
├── templates/
├── workflow/
├── interfaces/
├── contracts/
├── modules/
├── phases/
├── testing/
├── deployment/
└── decisions/

Each directory serves a distinct purpose, allowing AI to load only the documentation relevant to the current task.


---

22 Prompt Design Principles

The discussions consistently emphasized that effective prompts should:

Be single-objective.

Have explicit scope boundaries.

Define allowed and forbidden files.

Specify expected outputs and acceptance criteria.

Avoid repeating repository-wide context.

Reference existing documentation instead of embedding large explanations.

Be reusable through standardized templates.


---

23 Expected Benefits

Implementing the combined Repository Workflow Optimization Framework and Codex Token Usage Reduction Framework is expected to provide:

Significant reductions in prompt and context token consumption.

Lower AI operating costs.

Faster task completion.

Higher completion accuracy.

Reduced hallucinations.

Fewer unintended repository modifications.

Improved maintainability and onboarding.

Better modularity and separation of concerns.

Easier testing and debugging.

Greater consistency across AI-assisted development sessions.

Reusable documentation and prompt assets that compound efficiency over time.


Together, these discussions define an AI-first repository engineering methodology: structure the repository around modular components, capture architecture and rules in persistent documentation, constrain every AI task to the smallest practical scope, and standardize prompts so Codex operates with minimal context while producing predictable, high-quality results.
````

## File: robots.txt
````
User-agent: *
Allow: /

# Disallow admin and raw data folders
Disallow: /admin/
Disallow: /data/
Disallow: /assets/js/

# Sitemap location — update with your real domain
Sitemap: https://opportunitiesza.co.za/sitemap.xml
````

## File: sitemap-generator.js
````javascript
/**
 * OpportunitiesZA — Sitemap Generator
 * Run from project root: node sitemap-generator.js
 * Reads opportunities.json and guides.json, outputs sitemap.xml
 *
 * Usage from Termux:
 *   cd /storage/emulated/0/Download/opportunitiesza
 *   node sitemap-generator.js
 *   git add sitemap.xml && git commit -m "Update sitemap" && git push
 */

const fs   = require('fs');
const path = require('path');

// ── CONFIG — update BASE to your real domain ──────────────────
const BASE = 'https://opportunitiesza.co.za';
const DEFAULT_LASTMOD = '2026-05-01';
const TODAY = new Date().toISOString().slice(0, 10);

function isLiveOpportunity(o) {
  return !o.expired && (!o.closing_date || o.closing_date >= TODAY);
}

// ── STATIC PAGES ──────────────────────────────────────────────
const STATIC = [
  { loc: '/',                  priority: '1.0', freq: 'daily'   },
  { loc: '/learnerships/',     priority: '0.9', freq: 'daily'   },
  { loc: '/bursaries/',        priority: '0.9', freq: 'daily'   },
  { loc: '/internships/',      priority: '0.9', freq: 'daily'   },
  { loc: '/apprenticeships/',  priority: '0.9', freq: 'daily'   },
  { loc: '/opportunities/',    priority: '0.8', freq: 'daily'   },
  { loc: '/calendar/',         priority: '0.8', freq: 'daily'   },
  { loc: '/tools/',            priority: '0.8', freq: 'weekly'  },
  { loc: '/tools/eligibility/',priority: '0.7', freq: 'monthly' },
  { loc: '/tools/scam/',       priority: '0.7', freq: 'monthly' },
  { loc: '/tools/checklist/',  priority: '0.7', freq: 'monthly' },
  { loc: '/tools/bursary-checker/', priority: '0.7', freq: 'monthly' },
  { loc: '/seta-guides/',      priority: '0.8', freq: 'weekly'  },
  { loc: '/career-advice/',    priority: '0.7', freq: 'weekly'  },
  { loc: '/provinces/',        priority: '0.7', freq: 'weekly'  },
  { loc: '/about/',            priority: '0.4', freq: 'monthly' },
  // Province sub-pages
  { loc: '/learnerships/gauteng/',       priority: '0.7', freq: 'weekly' },
  { loc: '/learnerships/western-cape/',  priority: '0.7', freq: 'weekly' },
  { loc: '/learnerships/kwazulu-natal/', priority: '0.7', freq: 'weekly' },
  { loc: '/learnerships/eastern-cape/',  priority: '0.7', freq: 'weekly' },
  { loc: '/bursaries/gauteng/',          priority: '0.7', freq: 'weekly' },
  { loc: '/bursaries/western-cape/',     priority: '0.7', freq: 'weekly' },
  { loc: '/internships/gauteng/',        priority: '0.7', freq: 'weekly' },
  { loc: '/internships/western-cape/',   priority: '0.7', freq: 'weekly' },
];

// ── LOAD DATA FILES ───────────────────────────────────────────
function loadJSON(file) {
  try {
    return JSON.parse(fs.readFileSync(path.join(__dirname, file), 'utf8'));
  } catch (e) {
    console.warn(`⚠️  Could not load ${file}: ${e.message}`);
    return [];
  }
}

const opps   = loadJSON('data/opportunities.json');
const guides = loadJSON('data/guides.json');

// ── BUILD XML ─────────────────────────────────────────────────
function escapeXml(value) {
  return String(value)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&apos;');
}

function urlEntry({ loc, priority, freq, lastmod }) {
  return `  <url>
    <loc>${escapeXml(BASE + loc)}</loc>
    <lastmod>${lastmod || DEFAULT_LASTMOD}</lastmod>
    <changefreq>${freq || 'weekly'}</changefreq>
    <priority>${priority || '0.6'}</priority>
  </url>`;
}

let urls = [];

// Static pages
STATIC.forEach(p => urls.push(urlEntry(p)));

// Opportunity detail pages
const catPluralMap = {
  learnership: 'learnerships',
  internship: 'internships',
  bursary: 'bursaries',
  apprenticeship: 'apprenticeships',
  tvet: 'tvet-pathways',
};

opps
  .filter(isLiveOpportunity)
  .forEach(o => {
    const catPlural = catPluralMap[o.category] || (o.category + 's');
    urls.push(urlEntry({
      loc:     `/${catPlural}/${o.slug}`,
      priority: o.featured ? '0.8' : '0.7',
      freq:    'weekly',
      lastmod:  o.updated_date || DEFAULT_LASTMOD,
    }));
  });

// Guide pages
const guideCatMap = {
  'seta-guides':    'seta-guides',
  'career-advice':  'career-advice',
};

guides.forEach(g => {
  const cat = guideCatMap[g.category] || 'seta-guides';
  urls.push(urlEntry({
    loc:     `/${cat}/${g.slug}`,
    priority: '0.8',
    freq:    'monthly',
    lastmod:  g.updated_date || DEFAULT_LASTMOD,
  }));
});

// ── WRITE SITEMAP ─────────────────────────────────────────────
const xml = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.sitemaps.org/schemas/sitemap/0.9
        http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd">

${urls.join('\n\n')}

</urlset>
`;

fs.writeFileSync(path.join(__dirname, 'sitemap.xml'), xml, 'utf8');

console.log(`\n✅ sitemap.xml generated successfully`);
console.log(`   Static pages:     ${STATIC.length}`);
console.log(`   Opportunity pages: ${opps.filter(isLiveOpportunity).length}`);
console.log(`   Guide pages:       ${guides.length}`);
console.log(`   Total URLs:        ${urls.length}`);
console.log(`\n📋 Next steps:`);
console.log(`   git add sitemap.xml`);
console.log(`   git commit -m "Update sitemap"`);
console.log(`   git push\n`);
````

## File: sitemap.xml
````xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.sitemaps.org/schemas/sitemap/0.9
        http://www.sitemaps.org/schemas/sitemap/0.9/sitemap.xsd">

  <url>
    <loc>https://opportunitiesza.co.za/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>daily</changefreq>
    <priority>1.0</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/learnerships/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>daily</changefreq>
    <priority>0.9</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/bursaries/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>daily</changefreq>
    <priority>0.9</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/internships/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>daily</changefreq>
    <priority>0.9</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/apprenticeships/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>daily</changefreq>
    <priority>0.9</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/opportunities/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>daily</changefreq>
    <priority>0.8</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/calendar/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>daily</changefreq>
    <priority>0.8</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/tools/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.8</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/tools/eligibility/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/tools/scam/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/tools/checklist/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/tools/bursary-checker/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/seta-guides/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.8</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/career-advice/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/provinces/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/about/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.4</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/learnerships/gauteng/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/learnerships/western-cape/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/learnerships/kwazulu-natal/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/learnerships/eastern-cape/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/bursaries/gauteng/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/bursaries/western-cape/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/internships/gauteng/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/internships/western-cape/</loc>
    <lastmod>2026-05-01</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/learnerships/eskom-engineering-learnership-2026</loc>
    <lastmod>2026-06-03</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.8</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/internships/absa-graduate-internship-2026</loc>
    <lastmod>2026-04-05</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/bursaries/nsfas-bursary-2026</loc>
    <lastmod>2026-06-03</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.8</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/learnerships/mict-seta-ict-learnership-2026</loc>
    <lastmod>2026-04-08</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/apprenticeships/transnet-apprenticeship-2026</loc>
    <lastmod>2026-04-10</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/learnerships/shoprite-retail-learnership-2026</loc>
    <lastmod>2026-04-12</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/internships/kzn-health-internship-2026</loc>
    <lastmod>2026-04-14</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/bursaries/sasol-bursary-engineering-2026</loc>
    <lastmod>2026-06-03</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.8</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/apprenticeships/merseta-boilermaker-apprenticeship-2026</loc>
    <lastmod>2026-04-16</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/internships/old-mutual-graduate-internship-2026</loc>
    <lastmod>2026-04-18</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/internships/standard-bank-yes-internship-2026</loc>
    <lastmod>2026-04-20</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/learnerships/capitec-bank-learnership-2026</loc>
    <lastmod>2026-06-03</lastmod>
    <changefreq>weekly</changefreq>
    <priority>0.7</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/seta-guides/what-is-a-seta</loc>
    <lastmod>2026-04-20</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/seta-guides/what-is-a-learnership</loc>
    <lastmod>2026-04-02</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/seta-guides/how-to-apply-for-learnerships</loc>
    <lastmod>2026-04-03</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/seta-guides/learnership-scams</loc>
    <lastmod>2026-06-03</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/seta-guides/nqf-levels-explained</loc>
    <lastmod>2026-04-05</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/seta-guides/nsfas-2026</loc>
    <lastmod>2026-04-06</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>

  <url>
    <loc>https://opportunitiesza.co.za/career-advice/cv-for-learnership</loc>
    <lastmod>2026-04-07</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>

</urlset>
````

## File: validate-content.js
````javascript
#!/usr/bin/env node
/**
 * Validates that canonical opportunity data, the runtime loader in index.html,
 * and generated sitemap.xml agree on live opportunity slugs.
 */

const fs = require('fs');
const path = require('path');

const ROOT = __dirname;
const BASE = 'https://opportunitiesza.co.za';
const TODAY = new Date().toISOString().split('T')[0];
const DATA_PATH = 'data/opportunities.json';

const catPluralMap = {
  learnership: 'learnerships',
  internship: 'internships',
  bursary: 'bursaries',
  apprenticeship: 'apprenticeships',
  tvet: 'tvet-pathways',
};

function read(file) {
  return fs.readFileSync(path.join(ROOT, file), 'utf8');
}

function loadJSON(file) {
  return JSON.parse(read(file));
}

function isLiveOpportunity(o) {
  return !o.expired && (!o.closing_date || o.closing_date >= TODAY);
}

function opportunityPath(o) {
  const catPlural = catPluralMap[o.category] || `${o.category}s`;
  return `/${catPlural}/${o.slug}`;
}

function difference(left, right) {
  return [...left].filter(value => !right.has(value)).sort();
}

function assertNoDiff(label, leftName, left, rightName, right) {
  const missing = difference(left, right);
  if (missing.length) {
    console.error(`❌ ${label}: ${leftName} has entries missing from ${rightName}:`);
    missing.forEach(value => console.error(`   - ${value}`));
    process.exitCode = 1;
  }
}

const indexHTML = read('index.html');
if (!indexHTML.includes("const DATA_BASE='/data'") || !indexHTML.includes("fetchJSON('opportunities.json')")) {
  console.error(`❌ index.html is not configured to load opportunities from /data/opportunities.json.`);
  process.exit(1);
}
if (/const\s+OPPS\s*=\s*\[/.test(indexHTML)) {
  console.error('❌ index.html still contains a hardcoded OPPS array.');
  process.exit(1);
}

const opportunities = loadJSON(DATA_PATH);
const livePaths = new Set(opportunities.filter(isLiveOpportunity).map(opportunityPath));
const liveSlugs = new Set(opportunities.filter(isLiveOpportunity).map(o => o.slug));

const duplicateSlugs = [...liveSlugs].filter(slug => opportunities.filter(o => isLiveOpportunity(o) && o.slug === slug).length > 1);
if (duplicateSlugs.length) {
  console.error('❌ Duplicate live opportunity slugs found in data/opportunities.json:');
  duplicateSlugs.forEach(slug => console.error(`   - ${slug}`));
  process.exitCode = 1;
}

const sitemap = read('sitemap.xml');
const sitemapPaths = new Set(
  [...sitemap.matchAll(/<loc>(.*?)<\/loc>/g)]
    .map(match => match[1])
    .filter(loc => loc.startsWith(BASE))
    .map(loc => loc.slice(BASE.length))
    .filter(loc => /^\/(learnerships|internships|bursaries|apprenticeships|tvet-pathways)\/[^/]+$/.test(loc))
);

assertNoDiff('Opportunity sitemap sync', 'data/opportunities.json', livePaths, 'sitemap.xml', sitemapPaths);
assertNoDiff('Opportunity sitemap sync', 'sitemap.xml', sitemapPaths, 'data/opportunities.json', livePaths);

if (process.exitCode) process.exit(process.exitCode);
console.log(`✅ Content validation passed: ${liveSlugs.size} live opportunity slugs match index.html loader and sitemap.xml.`);
````

