# FORENSIC AUDIT - MYLOTTOEXPERT / WIREAUDIT-2
# Date: 2026-06-01
# Scope: Report Only. No code was changed.
# Files Audited:
#   CoreMylottoexpert 06 01 26 AM.txt   (Core PHP, 46560 lines, 730 functions, 2.9 MB)
#   Skai Simpler 06 01 26 AM.txt        (SKAI PHP+JS, 63036 lines, 1549 functions, 2.8 MB)
#   JS Mylottoexpert 06 01 26 AM.txt    (Frontend JS, 12263 lines, 804 functions, 595 KB)
#   CSS Mylottoexpert 06 01 26 AM.txt   (CSS, 28 lines minified, 566 KB)
#   MyLottoCRON 05 31 26.txt            (Cron PHP, 1799 lines, 53 functions, 83 KB)
#   my tables for SKAI and learning.txt (DB reference, 654 lines, 33 KB)
# Encoding: UTF-8, ASCII-safe output. No special glyphs.

---

## SECTION 1 - CRON FILE (MyLottoCRON 05 31 26.txt)

---

### CRON-1 [DEAD CODE] lecron_out() is defined but never called

  File:  MyLottoCRON 05 31 26.txt
  Lines: 68-76

  The function lecron_out() formats a message for CLI or web and is a well-written
  utility. However it is never called anywhere in the file. All output goes through
  direct echo statements or lecron_log(). The function can be removed or, if intentional
  for future use, should be documented as a reserved helper.

  Impact: None at runtime. Dead code adds maintenance confusion.

---

### CRON-2 [DEAD CONSTANT] LOTTOEXPERT_POST_DRAW_CRON_DEFAULT_LIMIT is never used

  File:  MyLottoCRON 05 31 26.txt
  Line:  26

  The constant LOTTOEXPERT_POST_DRAW_CRON_DEFAULT_LIMIT is defined as 0.
  It is never referenced in any subsequent code. lecron_resolve_limit() uses its
  own inline default of 500 and the constant LOTTOEXPERT_POST_DRAW_CRON_MAX_LIMIT,
  but not this constant.

  Impact: None at runtime. Misleading documentation value.

---

### CRON-3 [VERSION INCONSISTENCY] Three conflicting version strings

  File:  MyLottoCRON 05 31 26.txt
  Lines: 3 (header comment), 24 (constant), 1673 (profile JSON)

  The file header says "Version: V6 2026-05-28".
  The constant LOTTOEXPERT_POST_DRAW_CRON_VERSION says 'v7_2026_05_28_unified_evidence'.
  The shared profile JSON version field says 'cron_v7_phase42_shared_profile_refresh'.

  Three different version identifiers exist in the same file. The header was not
  updated when the constant was bumped from v6 to v7.

  Impact: Log tracing and support are confused by mismatched versions.

---

### CRON-4 [DEAD BRANCH] lecron_update_saved_score() if/else branches are identical

  File:  MyLottoCRON 05 31 26.txt
  Lines: 953-957

  Code:
    if (is_int($value) || is_float($value)) {
        $sets[] = $db->quoteName($col) . ' = ' . $db->quote((string)$value);
    } else {
        $sets[] = $db->quoteName($col) . ' = ' . $db->quote((string)$value);
    }

  Both branches execute the same statement. The condition was presumably written to
  emit a bare numeric literal in the SQL for numeric types (e.g. col = 3.14 instead
  of col = '3.14'). The intended else branch was never completed and both branches
  now do identical work. This is also present in lecron_update_assignment().

  Impact: Numeric columns are always quoted as strings in the SQL. Joomla's underlying
  PDO driver coerces types on bind anyway, but the intent is not honored.

---

### CRON-5 [HARDCODED MAGIC WEIGHTS] Learning score formula uses undocumented constants

  File:  MyLottoCRON 05 31 26.txt
  Line:  1518

  Code:
    $learningScore = ($mainHits * 100.0) + ($extraHits * 35.0)
                   + ((int)$rankMetrics['top10_hits'] * 6.0)
                   + ((int)$rankMetrics['top20_hits'] * 3.0);

  The multipliers 100, 35, 6, and 3 are hardcoded magic numbers with no explanation,
  no named constants, and no reference to a design document. If the scoring model
  changes, developers must hunt for these values.

  Additionally, top10_hits and top20_hits can overlap (a hit in the top 10 is also
  in the top 20). A number that appears in both gets credited 6 + 3 = 9 bonus points.
  This double-credit may be intentional but is not documented.

  Impact: Maintenance risk. Scoring formula is opaque and the double-credit behavior
  may produce unexpected score inflation for small predicted sets.

---

### CRON-6 [STUB METRICS] lecron_insert_learning_event() writes zeros for all lift metrics

  File:  MyLottoCRON 05 31 26.txt
  Lines: 1115-1130 (approx)

  The following metric columns are hardcoded to 0.0:
    top5_containment
    top10_containment
    mean_reciprocal_winner_rank
    winner_rank_lift_random
    winner_rank_lift_frequency
    winner_rank_lift_recency
    winner_rank_lift_skip
    promotion_help_rate
    promotion_harm_rate
    protected_core_regression_rate

  These appear to be advanced learning signals that were never implemented in the
  cron path. The columns exist in the schema (based on the insert column list) but
  are always written as zero from the cron.

  Impact: Any analytics or reporting that reads these columns from cron-sourced events
  will see all zeros and draw incorrect conclusions about lift performance.

---

### CRON-7 [REDUNDANT DATA] Learning event stores the same JSON in three columns

  File:  MyLottoCRON 05 31 26.txt
  Lines: approx 1140-1152

  The values array maps score evaluation_json to:
    'event_json'    => lecron_json_encode($score['evaluation_json'])
    'payload_json'  => lecron_json_encode($score['evaluation_json'])
    'event_data'    => lecron_json_encode($score['evaluation_json'])

  All three columns receive the identical JSON payload. This wastes storage and
  creates confusion about which column is authoritative.

  Impact: Storage overhead per learning event. Schema confusion.

---

### CRON-8 [DRY-RUN GAP] Dry-run skips duplicate check in lecron_insert_learning_event()

  File:  MyLottoCRON 05 31 26.txt
  Lines: 1163-1166

  The function returns true early when dryRun is true, before calling
  lecron_learning_event_exists(). In real mode the duplicate check prevents
  inserting the same event twice. In dry-run mode there is no such check,
  so dry-run always reports "would insert" even for rows that already have
  a learning event.

  Impact: Dry-run summary over-counts learning_events. The count shown in dry-run
  output is not an accurate preview of what a live run would do.

---

### CRON-9 [SECURITY / INFORMATION DISCLOSURE] Cron JSON summary exposed over HTTP

  File:  MyLottoCRON 05 31 26.txt
  Lines: 1680-1695, 1755-1760

  When invoked via HTTP (not CLI), the cron emits a JSON response that includes
  an 'items' array. Each item contains:
    game_id, draw_date, main_hits, extra_hits, learning_score, lane, saved_row_id

  This is internal database data. While the token authentication prevents unauthorized
  access, anyone who holds the token (including a compromised admin credential)
  can enumerate internal prediction scoring data via a simple HTTP GET.

  The web invocation path also requires that the server serve the cron file directly
  from a web-accessible location, which is a general security concern regardless of
  token validation.

  Recommendation: Prefer CLI-only invocation. If HTTP invocation must be supported,
  omit the 'items' array from the HTTP response and return only summary counts.

---

### CRON-10 [PERFORMANCE RISK] Candidate query uses LIKE '%keyword%' on multiple columns

  File:  MyLottoCRON 05 31 26.txt
  Lines: 820-840 (approx, inside lecron_find_candidates)

  The candidate selection query builds a WHERE clause that applies LIKE '%skai%',
  LIKE '%hive%', LIKE '%ai_prediction%', LIKE '%core_ai%', LIKE '%mcmc%',
  LIKE '%skip%', LIKE '%hit%' to each of 7 source columns
  (source, origin_source, analysis_method_key, model_family, batch_type,
  focus_profile, hive_intake_mode).

  Each leading-wildcard LIKE forces a full column scan. With 7 columns and 7 patterns
  that is up to 49 LIKE predicates in the OR. On a large user_saved_numbers table
  this query will not use indexes efficiently.

  Additionally, the pattern '%hit%' is very broad and could match any column value
  that happens to contain the substring 'hit' (e.g. 'prohibited', 'exhibit', 'white').

  Impact: Slow cron runs on large tables. False-positive candidate rows from broad
  LIKE patterns.

---

### CRON-11 [HARDCODED CONFIG THRESHOLDS] Confidence thresholds in lecron_refresh_shared_skai_profile()

  File:  MyLottoCRON 05 31 26.txt
  Lines: 1660-1666 (approx)

  The confidence formula uses hardcoded divisors:
    min(0.95, ($runCount / 60.0) * 0.45 + ($drawCount / 10.0) * 0.35 + ($userCount / 3.0) * 0.20)

  Status promotion thresholds are also hardcoded inline:
    'ready':  userCount >= 3, runCount >= 30, drawCount >= 5, confidence >= 0.55
    'strong': userCount >= 5, runCount >= 80, drawCount >= 12, confidence >= 0.72

  None of these are named constants. They cannot be tuned without editing the function.

  Impact: Maintenance risk. Tuning requires direct code edits with no central reference.

---

## SECTION 2 - SKAI FILE (Skai Simpler 06 01 26 AM.txt)

---

### SKAI-1 [HIGH SEVERITY] Hardcoded absolute server paths with username

  File:  Skai Simpler 06 01 26 AM.txt
  Lines: 1329-1335

  Code:
    $masterJsonPath = '/home/oscara/web/lottoexpert.net/public_html/lottery_skip_config.json';
    $dailyJsonPath  = '/home/oscara/web/lottoexpert.net/public_html/dailylotteries.json';
    $dailyBridgePath = '/home/oscara/web/lottoexpert.net/public_html/skai_daily_bridge.php';

  These three paths are hardcoded to the literal server home directory including the
  server account username 'oscara'. The rest of the codebase uses JPATH_ROOT
  consistently to construct paths.

  This will break silently on any environment other than the production server
  (staging, local dev, or a migrated host). The paths also disclose the server
  account name.

  The fix is to replace all three with JPATH_ROOT . '/filename.json' (or .php).

  Impact: SKAI fails to load daily lottery config on any non-production environment.
  Staging and local testing are broken unless the /home/oscara path exists.

---

### SKAI-2 [SECURITY] CSP allows 'unsafe-inline' and 'unsafe-eval' in script-src

  File:  Skai Simpler 06 01 26 AM.txt
  Lines: 194-196

  The Content-Security-Policy header includes:
    'unsafe-inline' and 'unsafe-eval' in script-src and script-src-elem

  The inline comment acknowledges this ("Keep inline/eval allowances for Sorcerer &
  current inline scripts"). However these two directives completely neutralize the XSS
  protection that CSP is meant to provide. An attacker who injects any inline script
  or evaluates code via eval() can execute arbitrary JavaScript.

  There is no report-uri or report-to directive, so CSP violations are not logged.

  The correct fix is to migrate all inline scripts to external files (removing
  unsafe-inline) and replace eval() usages with explicit function calls (removing
  unsafe-eval). If Sorcerer prevents this, a nonce-based policy is a viable
  intermediate step.

  Impact: CSP header provides no XSS defense in its current form.

---

### SKAI-3 [SECURITY] display_errors is enabled globally when site debug is on

  File:  Skai Simpler 06 01 26 AM.txt
  Lines: 158-163

  Code:
    if ($isDebug) {
        ini_set('display_errors', '1');
        ini_set('display_startup_errors', '1');
        error_reporting(E_ALL);
    }

  When the Joomla site debug flag is enabled, all errors are printed to the page
  for every visitor, not just administrators. Error messages can reveal file paths,
  database schema details, and internal logic.

  The debug check should also verify that the current user is a super admin before
  enabling display_errors.

  Impact: Information disclosure to regular users and unauthenticated visitors when
  debug mode is on.

---

### SKAI-4 [SIDE EFFECT] error_log path is set unconditionally on every page load

  File:  Skai Simpler 06 01 26 AM.txt
  Line:  165

  Code:
    ini_set('error_log', JPATH_ADMINISTRATOR . '/logs/skai_error.log');

  This overrides PHP's error_log setting globally for the entire request, on every
  page load that includes this file, regardless of context. If another component
  has already set a different error_log path, this will silently change it.

  Impact: Low severity. Can cause confusion when diagnosing errors from other
  components in the same request.

---

### SKAI-5 [DEAD CODE / DUPLICATE CONSTANTS] SKAI2 constants defined twice

  File:  Skai Simpler 06 01 26 AM.txt
  Lines: 173-184 (first block), 7505-7516 (second block)

  The same set of SKAI2_ constants is guarded and defined in two separate blocks.
  The if (!defined(...)) guards prevent actual redefinition errors, but the duplication
  suggests copy-paste from an earlier version and creates two code locations that
  must be kept in sync.

  Constants affected:
    SKAI2_OVERFIT_MIN_SAMPLE, SKAI2_OVERFIT_SIGNAL_BASELINE,
    SKAI2_CLUSTER_PROXIMITY_THRESHOLD, SKAI2_MAX_SKIP_CAP,
    SKAI2_LEARNING_MAX_CAP, SKAI2_LEARNING_MIN_CAP,
    SKAI2_MIN_EVIDENCE_DRAWS

  Impact: Maintenance risk. A change to a constant value must be made in two places.

---

### SKAI-6 [BUG RISK] file_get_contents() called without existence guard

  File:  Skai Simpler 06 01 26 AM.txt
  Line:  17501

  Code:
    $masterRaw = \file_get_contents($masterJsonPath);

  At this point $masterJsonPath is '/home/oscara/web/...' (see SKAI-1). There is no
  is_file() or file_exists() check before this call. If the file is missing, PHP
  emits a warning and file_get_contents() returns false. The subsequent json_decode()
  on false returns null. Depending on how the caller uses the result, this may
  silently produce incorrect behavior with no logged error.

  Impact: Silent failure when config file is missing. PHP warning emitted in logs.

---

### SKAI-7 [PERFORMANCE / CACHING] Inlined CSS animation and JS bootstrapped on every request

  File:  Skai Simpler 06 01 26 AM.txt
  Lines: 109-145 (approx, addStyleDeclaration and addScriptDeclaration blocks)

  Several kilobytes of CSS keyframe animations and JS initialization code are
  injected inline into every page response via addStyleDeclaration() and
  addScriptDeclaration(). Inline content cannot be cached by the browser.

  Moving these blocks into external .css and .js files (which the file already
  attempts to do for other assets at lines 99-108) would allow the browser to
  cache them across requests.

  Impact: Increased page payload on every load. No browser caching benefit.

---

## SECTION 3 - CORE PHP FILE (CoreMylottoexpert 06 01 26 AM.txt)

---

### CORE-1 [MAINTAINABILITY] Single-file complexity: 46560 lines, 730 functions

  File:  CoreMylottoexpert 06 01 26 AM.txt

  The core file is 2.9 MB and contains 730 functions in a single PHP include file.
  This exceeds any reasonable single-file complexity threshold. Symptoms:

    - Three unresolved TODO comments at lines 5233, 33615, and 39853.
    - Function names use a mix of styles: mylottoexpert, mylottoexpertV2, 
      mylottoexpertV55, mylottoexpertV122, mylottoexpertV135, mleV102, 
      and others. Versioned prefixes suggest accumulated layering without cleanup.
    - 637 uses of is_array/is_string/is_int/is_numeric type guards, indicating
      defensive programming against uncertain input types throughout.

  Impact: Any modification risks unintended side effects. Testing requires extensive
  manual inspection because the file has no automated test coverage visible from
  the source.

---

### CORE-2 [DEAD BRANCH] Identical if/else branches in value quoting loop

  File:  CoreMylottoexpert 06 01 26 AM.txt
  Multiple locations (same pattern as CRON-4)

  The same is_int/is_float dead branch pattern appears in multiple update functions
  throughout the Core file. Both branches call $db->quote((string)$value), making
  the conditional meaningless.

  Impact: Same as CRON-4. Numeric values are quoted as strings rather than emitted
  as raw SQL literals.

---

### CORE-3 [INCOMPLETE FEATURES] Three unresolved TODO comments

  File:  CoreMylottoexpert 06 01 26 AM.txt
  Lines: 5233, 33615, 39853

  Line 5233:   // TODO: Load from lottery_skip_config.json or database
  Line 33615:  // [TODO] Populate picks when UI/ML exposes them.
  Line 39853:  // TODO: Remove this variable in a future version once all legacy code is updated.

  These represent incomplete or deferred work. The variable referenced at line 39853
  may be accumulating as dead code until removal is completed.

  Impact: Undetermined. The feature gaps at 5233 and 33615 may affect lottery config
  loading and pick population respectively.

---

### CORE-4 [SECURITY NOTE] Admin operation endpoints rely on Joomla CSRF token alone

  File:  CoreMylottoexpert 06 01 26 AM.txt
  Lines: approx 15311, 24980

  Admin operations (rollback, quarantine, repair) are gated by Joomla session form
  token checks. This is the correct approach for Joomla. However rollback snapshot
  files are written to a directory using a timestamp-based token:

    $token = date('Ymd_His') . '_' . substr(sha1(json_encode($ids) . microtime(true)), 0, 10);
    $path = $dir . '/rollback_' . $token . '.json';

  The snapshot directory path ($dir) is not shown in the sampled lines. If $dir
  resolves to a web-accessible path, the rollback snapshot JSON files (which contain
  full database row data) could be read by anyone who can guess or enumerate
  the filename.

  Recommendation: Verify that the rollback snapshot directory is outside the
  web root, or that the directory has a .htaccess / nginx rule denying direct access.

---

## SECTION 4 - JAVASCRIPT FILE (JS Mylottoexpert 06 01 26 AM.txt)

---

### JS-1 [BUG] armFailSafe() is a no-op -- recovery overlay never triggers automatically

  File:  JS Mylottoexpert 06 01 26 AM.txt
  Lines: 6, 28-31, 41

  Code:
    var failTimer = null;
    ...
    function armFailSafe(ms){
      if (failTimer) { window.clearTimeout(failTimer); failTimer = null; }
      return;   // function exits here immediately
    }
    ...
    // called from show():
    armFailSafe(15000);

  The function clears any existing failTimer and then returns immediately.
  The `ms` parameter is never used. No new timeout is ever set.

  The original intent was clearly to set failTimer = window.setTimeout(showRecovery, ms)
  so that if the page fails to load within 15 seconds, the recovery UI (with
  'Stay on this page' and 'Reload workspace' buttons) would appear automatically.

  As written, the recovery UI only appears if showRecovery() is called explicitly
  from somewhere else in the code, which does not happen in the sampled code.

  Impact: Users who experience a loading failure see a stuck loading overlay with no
  automatic prompt to reload. The recovery buttons are present in the DOM but are
  never shown unless showRecovery() is called from outside.

---

### JS-2 [COMPLIANCE] ES5 confirmed in JS file

  File:  JS Mylottoexpert 06 01 26 AM.txt

  The loading experience module uses var, named function declarations, and no
  arrow functions, const, let, async, await, or Promise. ES5 compliance confirmed
  for this file.

---

## SECTION 5 - CSS FILE (CSS Mylottoexpert 06 01 26 AM.txt)

---

### CSS-1 [INFORMATIONAL] Single minified line, no issues identified

  File:  CSS Mylottoexpert 06 01 26 AM.txt

  The file is 28 lines, highly minified. No CSS syntax errors, illegal values,
  or security concerns were identified from the available content. The CSS
  follows the class naming conventions visible in the PHP/HTML output.

---

## SECTION 6 - DATABASE TABLES REFERENCE (my tables for SKAI and learning.txt)

---

### DB-1 [INFORMATIONAL] Reference file contains real data samples

  File:  my tables for SKAI and learning.txt

  This file is a database table reference containing column definitions and
  sample row data from production tables including:
    jos9d_mle_current_locked_settings
    jos9d_mle_duplicate_batch_quarantine_audit
    jos9d_mle_lottery_research_assignments
    jos9d_mle_lottery_research_queue

  The sample data rows include user_id values (e.g. 699, 438, 416) and production
  UUIDs, hashes, and timestamps. This file should not be stored in a public or
  shared repository as it contains identifiable production data.

  Impact: Privacy / data governance concern if this repository is shared publicly.

---

## SUMMARY TABLE

  ID       | Severity | Category              | File
  ---------|----------|-----------------------|-------------------------------
  CRON-1   | Low      | Dead Code             | MyLottoCRON
  CRON-2   | Low      | Dead Code             | MyLottoCRON
  CRON-3   | Low      | Version Inconsistency | MyLottoCRON
  CRON-4   | Low      | Dead Branch           | MyLottoCRON + Core
  CRON-5   | Medium   | Maintainability       | MyLottoCRON
  CRON-6   | Medium   | Incomplete Feature    | MyLottoCRON
  CRON-7   | Low      | Redundant Storage     | MyLottoCRON
  CRON-8   | Low      | Test Accuracy         | MyLottoCRON
  CRON-9   | Medium   | Security / Info Leak  | MyLottoCRON
  CRON-10  | Medium   | Performance           | MyLottoCRON
  CRON-11  | Low      | Maintainability       | MyLottoCRON
  SKAI-1   | High     | Broken Config Path    | Skai Simpler
  SKAI-2   | High     | Security / CSP        | Skai Simpler
  SKAI-3   | Medium   | Security / Info Leak  | Skai Simpler
  SKAI-4   | Low      | Side Effect           | Skai Simpler
  SKAI-5   | Low      | Dead Code / Duplicate | Skai Simpler
  SKAI-6   | Medium   | Bug Risk              | Skai Simpler
  SKAI-7   | Low      | Performance           | Skai Simpler
  CORE-1   | Medium   | Maintainability       | Core PHP
  CORE-2   | Low      | Dead Branch           | Core PHP
  CORE-3   | Low      | Incomplete Feature    | Core PHP
  CORE-4   | Medium   | Security Review       | Core PHP
  JS-1     | High     | Bug / UX              | JS Loading Experience
  JS-2     | Info     | Compliance OK         | JS Loading Experience
  CSS-1    | Info     | No Issues             | CSS
  DB-1     | Medium   | Data Governance       | DB Reference File

---

## PRIORITY ACTION LIST

  1. SKAI-1 (HIGH): Replace /home/oscara hardcoded paths with JPATH_ROOT equivalents.
     This is a one-line fix per path and unblocks all non-production environments.

  2. JS-1 (HIGH): Implement armFailSafe() to actually set a window.setTimeout.
     Without this the loading overlay recovery never fires automatically on failure.

  3. SKAI-2 (HIGH): Plan migration away from 'unsafe-inline' and 'unsafe-eval' in CSP.
     Immediate step: add a report-uri directive to start capturing violations.

  4. CRON-9 (MEDIUM): Remove the 'items' array from HTTP cron responses.
     Return only counts (scanned, scored, waiting, failed) when invoked via HTTP.

  5. CRON-6 (MEDIUM): Decide whether the stub zero-metrics are permanent or pending.
     If permanent, remove the columns. If pending, add a clear TODO and track them.

  6. SKAI-3 (MEDIUM): Restrict display_errors to admin users only, not all visitors.

  7. CRON-10 (MEDIUM): Add database indexes on source/analysis_method_key/batch_uuid
     columns in user_saved_numbers to support the candidate selection query.

  8. DB-1 (MEDIUM): Remove or redact real user IDs and production UUIDs from the
     reference file before making this repository visible to any outside party.

---

END OF AUDIT REPORT
