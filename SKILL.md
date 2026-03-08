---
name: max-watcher
version: 2.4.1
description: AI-powered continuous research monitor and intelligence aggregator
author: OpenClaw Research Team
tags:
  - research
  - ai
  - automation
  - monitoring
  - intelligence
platforms:
  - linux
  - darwin
dependencies:
  - python>=3.9
  - jq
  - curl
  - git
  - sqlite3
  - poetry
required_env_vars:
  - MAXWATCH_API_KEY
  - MAXWATCH_DB_PATH (default: ~/.maxwatch/data.db)
  - MAXWATCH_LOG_LEVEL (default: INFO)
optional_env_vars:
  - MAXWATCH_WEBHOOK_URL
  - MAXWATCH_SLACK_WEBHOOK
  - MAXWATCH_NOTIFY_EMAIL
  - MAXWATCH_CACHE_TTL (default: 3600)
  - MAXWATCH_MAX_WORKERS (default: 4)
---

# Max Watcher

AI-powered continuous research monitor that tracks sources, analyzes content, and delivers actionable intelligence.

## Purpose

Real-world use cases:
- Monitor arXiv for new papers matching specific research queries (e.g., "transformer architecture optimization" or "neurosymbolic AI")
- Track GitHub repositories for security vulnerabilities, new releases, or code patterns
- Watch Hacker News, Reddit, or specific Subreddits for trending discussions in your field
- Scrape and summarize competitor blog posts, whitepapers, or documentation
- Monitor conference proceedings (NeurIPS, ICML, CVPR) for relevant submissions
- Track funding announcements, grant opportunities, or research calls
- Aggregate mentions of your work across academic and social channels
- Build automated literature reviews from multiple sources
- Detect and summarize changes in API documentation or standards
- Monitor dataset releases fromKaggle, HuggingFace, or government sources

## Scope

Max Watcher provides these commands:

### Core Commands

```bash
# Add a new source to monitor
maxwatch add-source --type <type> --config <json>
maxwatch add-source arxiv --config '{"query":"machine learning fairness","categories":["cs.LG","cs.AI"],"max_results":50}'
maxwatch add-source github --config '{"owner":"torvalds","repo":"linux","watch_releases":true,"watch_issues":true,"labels":["security"]}'
maxwatch add-source rss --config '{"url":"https://example.com/feed","category":"news"}'
maxwatch add-source webpage --config '{"url":"https://blog.example.com","css_selector":".post","extract_links":true}'
maxwatch add-source reddit --config '{"subreddit":"MachineLearning","keywords":["transformer","attention"],"include_comments":true}'
maxwatch add-source twitter --config '{"handle":"@OpenAI","track_mentions":true}'
maxwatch add-source api --config '{"endpoint":"https://api.example.com/v1/papers","method":"GET","auth_type":"bearer","data_path":"$.results"}'
maxwatch add-source slack --config '{"channel":"#research-alerts","workspace":"acme"}'
maxwatch add-source email --config '{"imap_server":"imap.example.com","mailbox":"INBOX","search_criteria":"UNSEEN"}'
maxwatch add-source discord --config '{"channel_id":"123456","guild_id":"789012","keywords":["paper","release"]}'
maxwatch add-source hackernews --config '{"max_items":100,"min_points":50,"query":"AI safety"}'
maxwatch add-source git --config '{"repo_path":"/path/to/local/repo","branch":"main","track_uncommitted":false}'

# Remove a source
maxwatch remove-source <source_id>

# List all sources
maxwatch list-sources [--status active|paused|error] [--type <type>]

# Test a source configuration
maxwatch test-source <source_id> [--verbose]

# Start watching (daemon mode)
maxwatch watch --daemon [--port <port>] [--interval <seconds>]
maxwatch watch --interval 300  # check every 5 minutes

# One-time fetch and analyze
maxwatch fetch --source <source_id> [--force] [--output json|markdown]
maxwatch fetch --all --output markdown > research_digest_$(date +%Y%m%d).md

# Search collected items
maxwatch search "<query>" [--source <source_id>] [--date ">2024-01-01"] [--min-score <0-1>]
maxwatch search "neurosymbolic" --source arxiv --date ">2024-01-01" --min-score 0.7

# Analyze with custom prompt
maxwatch analyze <item_id> --prompt "Summarize key contributions and limitations"
maxwatch analyze --batch <item_ids> --template "technical_summary"

# Generate reports
maxwatch report daily --format html --email user@example.com
maxwatch report weekly --format pdf --output ~/reports/weekly_$(date +%Y%U).pdf
maxwatch report monthly --trends --compare-previous

# Tag and categorize items
maxwatch tag <item_id> --add "high-priority,relevant"
maxwatch tag --bulk <item_ids> --remove "low-quality"

# Export data
maxwatch export --format csv --output data.csv --fields title,url,score,published_at
maxwatch export --format bibtex --output library.bib --query "transformer optimization"

# Manage AI models
maxwatch models list
maxwatch models set-default <model_name>  # e.g., claude-3.5-sonnet, gpt-4, local-llama2
maxwatch models configure <model_name> --api-key <key> --endpoint <url>

# System management
maxwatch status  # show running processes, queue size, last fetch times
maxwatch config show  # display current configuration
maxwatch config set <key> <value>  # e.g., maxwatch config set notification.email user@example.com
maxwatch logs --follow --since 1h
maxwatch health  # database integrity, API connectivity, disk space

# Database maintenance
maxwatch db vacuum
maxwatch db archive --older-than 90d
maxwatch db export --backup ~/backups/maxwatch_$(date +%Y%m%d_%H%M%S).db
```

## Detailed Work Process

### Initial Setup
1. Install: `pip install maxwatch` or `brew install maxwatch`
2. Initialize database: `maxwatch init`
3. Configure API keys: `export MAXWATCH_API_KEY="sk-..."`
4. Test installation: `maxwatch health`

### Adding Sources
Real workflow example for academic research monitoring:

```bash
# Add arXiv source for specific research area
maxwatch add-source arxiv --config '{
  "query": "large language model interpretability",
  "categories": ["cs.CL", "cs.AI", "cs.LG"],
  "exclude_categories": ["cs.IR"],
  "max_results": 100,
  "sort_by": "relevance",
  "date_range": "30d",
  "authors": ["Yoshua Bengio", "Chris Olah"],  # optional filter
  "exclude_predatory": true
}'

# Add GitHub source for key repositories
maxwatch add-source github --config '{
  "owner": "huggingface",
  "repo": "transformers",
  "watch_releases": true,
  "watch_issues": false,
  "watch_prs": true,
  "labels": ["breaking-change", "security"],
  "notify_on": ["release', 'security_alert']
}'

# Add RSS for research blogs
maxwatch add-source rss --config '{
  "url": "https://distill.pub/rss.xml",
  "category": "visualization",
  "extract_full_content": true,
  "summary_prompt": "Extract key research insights and methodology"
}'
```

### Fetch Cycle
1. Trigger: `maxwatch fetch --all --force`
2. Process:
   - Pull new items from each source
   - Deduplicate based on URL, title hash, or DOI
   - Score relevance using AI (0-1 scale)
   - Extract entities (authors, institutions, datasets, methods)
   - Generate summaries (configurable length: short/medium/detailed)
   - Apply classification tags based on content
   - Store in SQLite with metadata (source, fetched_at, raw_content, embeddings)
3. Rate limiting: Respect source-specific intervals (arXiv: 3s, GitHub: 1s)
4. Error handling: Retry 3x with exponential backoff, mark source as error after 5 consecutive failures

### Analysis and Filtering
```bash
# Find high-value items
maxwatch search "interpretability" --min-score 0.85 --since "7d" --sort score

# Batch analyze with custom prompt
maxwatch analyze --batch $(maxwatch search "attention mechanism" --min-score 0.8 | jq -r '.id') \
  --prompt "Identify novel contributions, methodology strengths, and potential limitations"

# Export to reference manager
maxwatch export --format bibtex --output ~/research/library.bib --query "survey" --since "2024-01-01"
```

### Notifications
Configure hooks:
```bash
# Webhook (e.g., to n8n, Zapier, custom app)
maxwatch config set webhook_url "https://hooks.example.com/research"

# Slack
maxwatch config set slack_webhook "https://hooks.slack.com/services/..."

# Email digest
maxwatch config set notification.email " researcher@example.com"
maxwatch report daily --format markdown --email

# Custom script
maxwatch config set notification.script "/path/to/notify.sh"
```

### Maintenance
```bash
# Daily
maxwatch fetch --all
maxwatch db archive --older-than 30d
maxwatch logs --since 1d > ~/logs/maxwatch_$(date +%Y%m%d).log

# Weekly
maxwatch report weekly --format html --email
maxwatch db vacuum

# Monthly
maxwatch models list  # check for updates
maxwatch health
```

## Golden Rules

1. **Always deduplicate before storing**: Use SHA256 of (title + source + published_date) to avoid duplicates across sources.

2. **Respect rate limits**: Default intervals: arXiv (3s), GitHub (1s), RSS (60s), Twitter (2s). Never override without `--force`.

3. **Never run `vacuum` during active fetch**: Schedule maintenance only when `maxwatch status` shows idle.

4. **Always back up before mass delete**: `maxwatch db export --backup` before `maxwatch db archive` or bulk tag removal.

5. **AI scoring must be deterministic**: Same content should produce same score (temperature=0, fixed seed).

6. **Privacy first**: Never store API responses with personal data. Use `maxwatch config set anonymize_pii true` if processing sensitive sources.

7. **Source isolation**: One source failure must not cascade. Each source runs in separate worker with independent error tracking.

8. **Capacity limits**: Enforce `MAXWATCH_MAX_WORKERS` to avoid OOM. Default 4, max 16.

9. **Content preservation**: Raw JSON responses stored separately from extracted fields. Enables re-analysis with new prompts.

10. **Authenticity validation**: For academic sources, verify DOI and crossref metadata. For GitHub, verify signatures on releases.

## Examples

### Use Case 1: Monitor Emerging AI Safety Research
```bash
# Setup
maxwatch add-source arxiv --config '{
  "query": "AI alignment OR AI safety OR reward hacking",
  "categories": ["cs.AI", "cs.LG", "cs.CY"],
  "max_results": 200
}'
maxwatch add-source twitter --config '{
  "handle": "@DeepMindSafety",
  "track_mentions": false
}'

# Daily workflow
maxwatch fetch --all
MAX_RELEVANT=$(maxwatch search "AI safety" --min-score 0.75 --since "1d" --output json | jq -r '.items[] | "\(.title) - \(.url)"')
echo "$MAX_RELEVANT" | mail -s "AI Safety Digest $(date +%Y-%m-%d)" team@lab.org

# When high-priority item found
maxwatch tag <item_id> --add "team-review,high-priority"
maxwatch analyze <item_id> --prompt "Write 3 discussion questions for lab meeting"
```

### Use Case 2: Competitor Intelligence
```bash
# Watch competitor's GitHub and blog
maxwatch add-source github --config '{
  "owner":"openai",
  "repo":"*",
  "watch_releases":true,
  "watch_commits":true,
  "keywords":["API","pricing","deprecation"]
}'
maxwatch add-source rss --config '{
  "url":"https://openai.com/blog/rss",
  "category":"announcements"
}'

# Alert on specific changes
maxwatch search "pricing" --since "1d" --output json | jq -r '.items[] | "\(.source_type): \(.title) \(.url)"' | \
  while read line; do
    curl -X POST -H "Content-type: application/json" \
      --data "{\"text\":\"$line\"}" "$SLACK_WEBHOOK"
  done
```

### Use Case 3: Literature Review Automation
```bash
# Add multiple databases
maxwatch add-source arxiv --config '{"query":"federated learning healthcare","categories":["cs.LG","cs.CR"]}'
maxwatch add-source rss --config '{"url":"https://medrxiv.org/rss","category":"medical"}'
maxwatch add-source api --config '{"endpoint":"https://api.semanticscholar.org/graph/v1/paper/search","method":"GET","params":{"query":"privacy-preserving ML","fields":"title,year,citationCount"},"auth_headers":{"x-api-key":"$SEMANTIC_SCHOLAR_KEY"}}'

# Weekly report generation
maxwatch report weekly --format markdown --template literature_review \
  --output ~/papers/lit_review_$(date +%Y%U).md

# Export to Zotero-compatible format
maxwatch export --format zotero --output ~/zotero/import.xml --query "federated learning" --since "90d"
```

### Use Case 4: Security Monitoring
```bash
# Monitor security-related GitHub repos
maxwatch add-source github --config '{
  "owner":"CVEProject",
  "repo":"cvelistV5",
  "watch_releases":true,
  "labels":["cve"],
  "cvss_threshold":7.0
}'

# Critical alert pipeline
CRITICAL=$(maxwatch search "CVSS >= 8.0" --since "6h" --output json)
if [ -n "$CRITICAL" ]; then
  echo "$CRITICAL" | jq -r '.items[] | "[\(.severity)] \(.title) (\(.cvss_score)) \(.url)"' > /tmp/critical_cves.txt
  pagerduty --service-key $PD_KEY --event-type trigger --description "New critical CVEs detected" /tmp/critical_cves.txt
  maxwatch tag --bulk $(jq -r '.items[].id' <<< "$CRITICAL") --add "critical,action-required"
fi
```

## Rollback Commands

### Source Management
```bash
# Disable problematic source (preserves data)
maxwatch config set source.<source_id>.enabled false

# Re-enable source
maxwatch config set source.<source_id>.enabled true

# Restore deleted source (from config history)
maxwatch config history | grep "<source_id>" | tail -1 | cut -d' ' -f2 | xargs maxwatch config restore

# Re-run failed fetch with increased timeout
maxwatch fetch --source <source_id> --timeout 120
```

### Data Recovery
```bash
# Restore from backup (full database)
maxwatch db restore ~/backups/maxwatch_20240115_120000.db

# Recover items accidentally tagged (within 30 days)
maxwatch tag <item_id> --remove "deleted"  # if soft-deleted via tag
maxwatch db recover --tag "deleted" --older-than 30d

# Re-fetch specific date range (e.g., if source had downtime)
maxwatch fetch --source arxiv --date-range "2024-03-01,2024-03-05" --force

# Undo bulk tag operation (requires transaction log)
maxwatch tag --bulk <item_ids> --undo --transaction-id <tx_id>
```

### Configuration Rollback
```bash
# View config history
maxwatch config history

# Revert to previous config (last 5 minutes)
maxwatch config rollback --minutes 5

# Restore specific config version
maxwatch config restore --version <timestamp>

# Reset to defaults (keep data)
maxwatch config reset --keep-data
```

### AI Model Failover
```bash
# If primary model fails, quickly switch to backup
maxwatch models set-default gpt-4  # primary
# When rate-limited or errors:
maxwatch models set-default claude-3.5-sonnet  # backup
# Re-analyze previously failed items
maxwatch analyze --failed --model claude-3.5-sonnet
```

### Emergency Procedures
```bash
# Stop all watchers immediately
maxwatch stop --all --graceful 0  # no wait

# Clear stuck queue (items stuck in "processing" > 1h)
maxwatch queue clear --stale 1h --dry-run  # preview
maxwatch queue clear --stale 1h  # execute

# Free disk space (archive old data locally, keep indexes)
maxwatch db archive --older-than 180d --compress
maxwatch logs rotate --size 100M --keep 30

# Full reset (WARNING: deletes all data)
maxwatch reset --confirm $(hostname)-$(date +%s)
```

## Verification Steps

After installation:
```bash
# Verify database
maxwatch health | grep "database.*OK"

# Test source connectivity
maxwatch test-source arxiv_1 | grep "status.*success"

# Check API quota (if applicable)
maxwatch models list | grep "quota.*remaining"

# Verify notification pipeline
maxwatch config show | grep -E "(webhook|slack|email)" | grep -v null
echo "test" | maxwatch notify --dry-run
```

During operation:
```bash
# Check active processes
maxwatch status | grep "workers.*[0-9]\+.*[1-9]"

# Verify deduplication
maxwatch search --source arxiv_1 | wc -l  # should match expected count

# Confirm AI scoring ran
maxwatch search --min-score 0.0 | head -1 | jq '.score'  # should be 0.0-1.0

# Validate exports
maxwatch export --format bibtex --output test.bib --query "test"
grep "@article" test.bib  # should contain entries
```

After updates:
```bash
# Schema migration check
maxwatch db check-migrations
maxwatch db version  # should show latest

# Re-fetch one source to ensure compatibility
maxwatch fetch --source <existing_source> --force
```

## Troubleshooting Common Issues

**Symptom**: "Connection refused" for source
- Check source URL/endpoint: `maxwatch config show | grep source.<source_id>`
- Test manually: `curl -I <url>` or `nc -zv <host> <port>`
- Verify API keys: `echo $MAXWATCH_API_KEY | wc -c` (should be > 20)
- Check rate limits: look at `last_fetched` in DB: `maxwatch db query "SELECT * FROM sources WHERE id=<source_id>"`
- Increase timeout: `maxwatch config set source.<source_id>.timeout 30`

**Symptom**: Low relevance scores (all < 0.1)
- Test AI model: `maxwatch models test --sample "machine learning paper about transformers"`
- Check prompt: `maxwatch config get scoring.prompt` - should be research-specific
- Verify embeddings: `maxwatch db query "SELECT COUNT(*) FROM embeddings"` - should be > 0
- Adjust threshold: `maxwatch search --min-score 0.05` temporarily to verify items are captured

**Symptom**: Duplicates appearing
- Run deduplication: `maxwatch db deduplicate --dry-run` preview
- Check dedupe keys: `maxwatch config get deduplication.fields` should include title, url_hash, doi
- Force re-deduplicate: `maxwatch db deduplicate --force`

**Symptom**: Database locked / performance slow
- Check for stuck processes: `maxwatch status | grep stuck`
- Kill zombies: `maxwatch stop --stale 2h`
- Optimize: `maxwatch db vacuum --analyze`
- Check disk space: `df -h ~/.maxwatch/`

**Symptom**: Notifications not sending
- Verify webhook: `curl -X POST -d '{"test":true}' $MAXWATCH_WEBHOOK_URL`
- Check logs: `maxwatch logs --level ERROR --since 10m | grep notify`
- Test notification: `maxwatch notify --test --to user@example.com`
- Confirm email config: `maxwatch config get notification.email` should be valid

**Symptom**: High memory usage
- Check worker count: `maxwatch config get max_workers` (default 4)
- Reduce: `maxwatch config set max_workers 2`
- Monitor per-source: `maxwatch db query "SELECT source_id, COUNT(*) FROM items GROUP BY source_id"` - uneven distribution?
- Restart: `maxwatch restart`

**Symptom**: Missing recent items
- Check source status: `maxwatch list-sources --status error`
- Check last fetch: `maxwatch db query "SELECT last_fetched FROM sources WHERE id=<source_id>"`
- Verify interval not too long: `maxwatch config get source.<source_id>.interval` (default varies)
- Force immediate: `maxwatch fetch --source <source_id> --force`

**Symptom**: AI model quota exceeded
- Check usage: `maxwatch models usage`
- Switch to backup: `maxwatch models set-default <backup_model>`
- Increase quota or add new API key: `maxwatch models add-key <provider> <new_key>`
- Reduce scoring: `maxwatch config set scoring.enabled false` temporarily

**Symptom**: Export format errors
- Verify template exists: `maxwatch templates list`
- Test with small set: `maxwatch export --format csv --limit 10 --output test.csv`
- Check field names: `maxwatch export --fields-list` to see available fields

## Advanced Configuration

### Custom Scoring Pipeline
```bash
# Override default AI scoring with custom prompt
maxwatch config set scoring.prompt "
  Given this research item: {{content}}
  Rate relevance to: {{query}}
  Score 0-1 where:
  0.0-0.3: unrelated
  0.4-0.6: tangentially related
  0.7-0.9: directly relevant
  1.0: exactly matches focus area
  Output only JSON: {'score': <float>, 'reason': '<one sentence>'}
"

# Use ensemble scoring (multiple models)
maxwatch config set scoring.ensemble true
maxwatch config set scoring.models ["gpt-4", "claude-3.5-sonnet"]
maxwatch config set scoring.strategy "average"  # or "max", "weighted"
```

### Webhook Integration Example
```json
{
  "url": "https://your-automation.example.com/webhook",
  "headers": {
    "Authorization": "Bearer $WEBHOOK_TOKEN",
    "Content-Type": "application/json"
  },
  "template": "research_alert",
  "events": ["high_score", "new_source_item", "daily_digest"]
}
```

### Custom Source Development
```bash
# Create custom Python source plugin
mkdir -p ~/.maxwatch/plugins/
cat > ~/.maxwatch/plugins/my_custom_source.py << 'EOF'
from maxwatch.plugin import SourcePlugin
class MySource(SourcePlugin):
    def fetch(self):
        # Implement fetch logic
        return items
    def parse(self, raw):
        # Return dict with: title, url, content, published_at, etc.
        return parsed_item
EOF

maxwatch plugins enable my_custom_source
maxwatch add-source custom --config '{"plugin":"my_custom_source","params":{"endpoint":"..."}}'
```

## Performance Tuning

For large-scale monitoring (1000+ sources):
- Increase database WAL: `maxwatch config set db.wal_mode true`
- Adjust connection pool: `maxwatch config set db.pool_size 20`
- Enable parallel fetching: `maxwatch config set fetch.parallel true`
- Use Redis cache for embeddings: `maxwatch config set cache.backend redis --cache-url redis://localhost:6379`

For memory-constrained environments:
- Disable embedding storage: `maxwatch config set embeddings.enabled false`
- Limit content storage: `maxwatch config set content.max_length 10000` (truncate after 10KB)
- Archive immediately: `maxwatch config set archive.immediate true`

## Security Considerations

- Never commit `~/.maxwatch/config.json` with API keys. Use env vars.
- Rotate API keys quarterly: `maxwatch models rotate-key <provider>`
- Enable audit logging: `maxwatch config set logging.audit true`
- Restrict DB access: `chmod 600 ~/.maxwatch/data.db`
- Sanitize webhook payloads: `maxwatch config set webhook.sanitize true` (removes raw content by default)

## Support and Contributing

- Report issues: `maxwatch doctor --report | tee ~/maxwatch_issue.txt`
- Debug mode: `MAXWATCH_LOG_LEVEL=DEBUG maxwatch fetch --all`
- Community: https://github.com/openclaw/maxwatch/discussions
- Plugin SDK: https://maxwatch.openclaw.dev/plugins
```