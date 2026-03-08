name: max-watcher
version: 2.4.1
description: Monitor de investigación continuo impulsado por IA y agregador de inteligencia
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
  - MAXWATCH_DB_PATH (predeterminado: ~/.maxwatch/data.db)
  - MAXWATCH_LOG_LEVEL (predeterminado: INFO)
optional_env_vars:
  - MAXWATCH_WEBHOOK_URL
  - MAXWATCH_SLACK_WEBHOOK
  - MAXWATCH_NOTIFY_EMAIL
  - MAXWATCH_CACHE_TTL (predeterminado: 3600)
  - MAXWATCH_MAX_WORKERS (predeterminado: 4)
---

# Max Watcher

Monitor de investigación continuo impulsado por IA que rastrea fuentes, analiza contenido y entrega inteligencia procesable.

## Propósito

Casos de uso reales:
- Monitorear arXiv para nuevos artículos que coincidan con consultas de investigación específicas (ej., "optimización de arquitectura transformer" o "IA neurosimbolica")
- Rastrear repositorios de GitHub para vulnerabilidades de seguridad, nuevos lanzamientos o patrones de código
- Observar Hacker News, Reddit o Subreddits específicos para discusiones trending en tu campo
- Extraer y resumir publicaciones de blog de competidores, white papers o documentación
- Monitorear actas de conferencias (NeurIPS, ICML, CVPR) para presentaciones relevantes
- Rastrear anuncios de financiación, oportunidades de subvenciones o llamadas de investigación
- Agregar menciones de tu trabajo en canales académicos y sociales
- Construir revisiones de literatura automatizadas desde múltiples fuentes
- Detectar y resumir cambios en documentación de API o estándares
- Monitorear lanzamientos de conjuntos de datos de Kaggle, HuggingFace o fuentes gubernamentales

## Alcance

Max Watcher proporciona estos comandos:

### Comandos Principales

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

## Proceso de Trabajo Detallado

### Configuración Inicial
1. Instalar: `pip install maxwatch` o `brew install maxwatch`
2. Inicializar base de datos: `maxwatch init`
3. Configurar claves API: `export MAXWATCH_API_KEY="sk-..."`
4. Probar instalación: `maxwatch health`

### Agregando Fuentes
Ejemplo de flujo de trabajo real para monitoreo de investigación académica:

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

### Ciclo de Obtención
1. Disparador: `maxwatch fetch --all --force`
2. Proceso:
   - Obtener nuevos elementos de cada fuente
   - Deduplicar basado en URL, hash de título, o DOI
   - Puntuación de relevancia usando IA (escala 0-1)
   - Extraer entidades (autores, instituciones, conjuntos de datos, métodos)
   - Generar resúmenes (longitud configurable: corto/medio/detallado)
   - Aplicar etiquetas de clasificación basadas en contenido
   - Almacenar en SQLite con metadatos (fuente, fetched_at, raw_content, embeddings)
3. Límite de tasa: Respetar intervalos específicos de fuente (arXiv: 3s, GitHub: 1s)
4. Manejo de errores: Reintentar 3x con retroceso exponencial, marcar fuente como error después de 5 fallos consecutivos

### Análisis y Filtrado
```bash
# Find high-value items
maxwatch search "interpretability" --min-score 0.85 --since "7d" --sort score

# Batch analyze with custom prompt
maxwatch analyze --batch $(maxwatch search "attention mechanism" --min-score 0.8 | jq -r '.id') \
  --prompt "Identify novel contributions, methodology strengths, and potential limitations"

# Export to reference manager
maxwatch export --format bibtex --output ~/research/library.bib --query "survey" --since "2024-01-01"
```

### Notificaciones
Configurar hooks:
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

### Mantenimiento
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

## Reglas de Oro

1. **Siempre desduplicar antes de almacenar**: Usar SHA256 de (title + source + published_date) para evitar duplicados entre fuentes.

2. **Respetar límites de tasa**: Intervalos por defecto: arXiv (3s), GitHub (1s), RSS (60s), Twitter (2s). Nunca anular sin `--force`.

3. **Nunca ejecutar `vacuum` durante una obtención activa**: Programar mantenimiento solo cuando `maxwatch status` muestre inactividad.

4. **Siempre hacer copia de seguridad antes de eliminación masiva**: `maxwatch db export --backup` antes de `maxwatch db archive` o eliminación de etiquetas en bloque.

5. **La puntuación de IA debe ser determinista**: El mismo contenido debe producir la misma puntuación (temperature=0, semilla fija).

6. **Privacidad primero**: Nunca almacenar respuestas de API con datos personales. Usar `maxwatch config set anonymize_pii true` si se procesan fuentes sensibles.

7. **Aislamiento de fuente**: Un fallo en una fuente no debe cascadear. Cada fuente se ejecuta en un worker separado con seguimiento de errores independiente.

8. **Límites de capacidad**: Aplicar `MAXWATCH_MAX_WORKERS` para evitar OOM. Predeterminado 4, máximo 16.

9. **Preservación de contenido**: Las respuestas JSON crudas se almacenan por separado de los campos extraídos. Permite re-análisis con nuevos prompts.

10. **Validación de autenticidad**: Para fuentes académicas, verificar DOI y metadatos de crossref. Para GitHub, verificar firmas en releases.

## Ejemplos

### Caso de Uso 1: Monitorear Investigación Emergente de Seguridad de IA
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

### Caso de Uso 2: Inteligencia de Competidores
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

### Caso de Uso 3: Automatización de Revisión de Literatura
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

### Caso de Uso 4: Monitoreo de Seguridad
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

## Comandos de Reversión

### Gestión de Fuentes
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

### Recuperación de Datos
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

### Reversión de Configuración
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

### Conmutación por Fallo de Modelo de IA
```bash
# If primary model fails, quickly switch to backup
maxwatch models set-default gpt-4  # primary
# When rate-limited or errors:
maxwatch models set-default claude-3.5-sonnet  # backup
# Re-analyze previously failed items
maxwatch analyze --failed --model claude-3.5-sonnet
```

### Procedimientos de Emergencia
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

## Pasos de Verificación

Después de la instalación:
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

Durante la operación:
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

Después de actualizaciones:
```bash
# Schema migration check
maxwatch db check-migrations
maxwatch db version  # should show latest

# Re-fetch one source to ensure compatibility
maxwatch fetch --source <existing_source> --force
```

## Solución de Problemas Comunes

**Síntoma**: "Connection refused" para fuente
- Verificar URL/endpoint de fuente: `maxwatch config show | grep source.<source_id>`
- Probar manualmente: `curl -I <url>` o `nc -zv <host> <port>`
- Verificar claves API: `echo $MAXWATCH_API_KEY | wc -c` (debería ser > 20)
- Revisar límites de tasa: mirar `last_fetched` en DB: `maxwatch db query "SELECT * FROM sources WHERE id=<source_id>"`
- Aumentar timeout: `maxwatch config set source.<source_id>.timeout 30`

**Síntoma**: Puntuaciones de relevancia bajas (todas < 0.1)
- Probar modelo de IA: `maxwatch models test --sample "machine learning paper about transformers"`
- Revisar prompt: `maxwatch config get scoring.prompt` - debería ser específico de investigación
- Verificar embeddings: `maxwatch db query "SELECT COUNT(*) FROM embeddings"` - debería ser > 0
- Ajustar umbral: `maxwatch search --min-score 0.05` temporalmente para verificar que se capturan elementos

**Síntoma**: Duplicados aparecen
- Ejecutar deduplicación: `maxwatch db deduplicate --dry-run` vista previa
- Revisar claves de dedupe: `maxwatch config get deduplication.fields` debería incluir title, url_hash, doi
- Forzar re-deduplicación: `maxwatch db deduplicate --force`

**Síntoma**: Base de datos bloqueada / rendimiento lento
- Buscar procesos bloqueados: `maxwatch status | grep stuck`
- Matar zombies: `maxwatch stop --stale 2h`
- Optimizar: `maxwatch db vacuum --analyze`
- Revisar espacio en disco: `df -h ~/.maxwatch/`

**Síntoma**: Notificaciones no se envían
- Verificar webhook: `curl -X POST -d '{"test":true}' $MAXWATCH_WEBHOOK_URL`
- Revisar logs: `maxwatch logs --level ERROR --since 10m | grep notify`
- Probar notificación: `maxwatch notify --test --to user@example.com`
- Confirmar configuración de email: `maxwatch config get notification.email` debería ser válido

**Síntoma**: Alto uso de memoria
- Revisar conteo de workers: `maxwatch config get max_workers` (predeterminado 4)
- Reducir: `maxwatch config set max_workers 2`
- Monitorear por fuente: `maxwatch db query "SELECT source_id, COUNT(*) FROM items GROUP BY source_id"` - distribución desigual?
- Reiniciar: `maxwatch restart`

**Síntoma**: Elementos recientes faltantes
- Revisar estado de fuente: `maxwatch list-sources --status error`
- Revisar última obtención: `maxwatch db query "SELECT last_fetched FROM sources WHERE id=<source_id>"`
- Verificar que intervalo no es muy largo: `maxwatch config get source.<source_id>.interval` (predeterminado varia)
- Forzar inmediato: `maxwatch fetch --source <source_id> --force`

**Síntoma**: Cuota de modelo de IA excedida
- Revisar uso: `maxwatch models usage`
- Cambiar a respaldo: `maxwatch models set-default <backup_model>`
- Aumentar cuota o agregar nueva clave API: `maxwatch models add-key <provider> <new_key>`
- Reducir puntuación: `maxwatch config set scoring.enabled false` temporalmente

**Síntoma**: Errores de formato de exportación
- Verificar que plantilla existe: `maxwatch templates list`
- Probar con conjunto pequeño: `maxwatch export --format csv --limit 10 --output test.csv`
- Revisar nombres de campo: `maxwatch export --fields-list` para ver campos disponibles

## Configuración Avanzada

### Pipeline de Puntuación Personalizado
```bash
# Anular puntuación de IA predeterminada con prompt personalizado
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

# Usar puntuación de conjunto (múltiples modelos)
maxwatch config set scoring.ensemble true
maxwatch config set scoring.models ["gpt-4", "claude-3.5-sonnet"]
maxwatch config set scoring.strategy "average"  # or "max", "weighted"
```

### Ejemplo de Integración de Webhook
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

### Desarrollo de Fuente Personalizada
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

## Ajuste de Rendimiento

Para monitoreo a gran escala (1000+ fuentes):
- Aumentar WAL de base de datos: `maxwatch config set db.wal_mode true`
- Ajustar pool de conexiones: `maxwatch config set db.pool_size 20`
- Habilitar obtención paralela: `maxwatch config set fetch.parallel true`
- Usar cache Redis para embeddings: `maxwatch config set cache.backend redis --cache-url redis://localhost:6379`

Para entornos con memoria limitada:
- Deshabilitar almacenamiento de embeddings: `maxwatch config set embeddings.enabled false`
- Limitar almacenamiento de contenido: `maxwatch config set content.max_length 10000` (truncar después de 10KB)
- Archivar inmediatamente: `maxwatch config set archive.immediate true`

## Consideraciones de Seguridad

- Nunca cometer `~/.maxwatch/config.json` con claves API. Usar variables de entorno.
- Rotar claves API trimestralmente: `maxwatch models rotate-key <provider>`
- Habilitar logging de auditoría: `maxwatch config set logging.audit true`
- Restringir acceso a DB: `chmod 600 ~/.maxwatch/data.db`
- Sanitizar payloads de webhook: `maxwatch config set webhook.sanitize true` (elimina contenido crudo por defecto)

## Soporte y Contribución

- Reportar issues: `maxwatch doctor --report | tee ~/maxwatch_issue.txt`
- Modo debug: `MAXWATCH_LOG_LEVEL=DEBUG maxwatch fetch --all`
- Comunidad: https://github.com/openclaw/maxwatch/discussions
- Plugin SDK: https://maxwatch.openclaw.dev/plugins