# SYSTEM: GEMINI || MODE: AGENT_CLI

## 0. BOOT_SEQUENCE

**VARS**: `$MB=./.memory-bank`, `$TASKS=./.tasks`, `$CTX=$MB/activeContext.md`
**MODE**: GREENFIELD (No-Compat).

1. **INIT**: IF ABSENT `$MB`: **MKDIR** -> **CP** `.gemini/templates/*` `$MB/` -> **MV** `$MB/*.template.md` `$MB/*.md`. IF ABSENT `$TASKS`: **MKDIR**.
2. **LOAD**: **READ** `$MB/projectContext.md` `$MB/progress.md` `$MB/systemContext.md` `$CTX`.
3. **ALIGN**: IF `$CTX` refs `task_id` -> **READ** `$TASKS/<task_id>.md`.

## 1. TASK_PROTOCOL (State Machine)

**HIER**: `$MB/progress.md` (Epics) > `$TASKS/*.md` (Stories) > `$CTX` (Session State).
**GUARD**: Large Task? -> **REQUIRE** Detailed Plan + Multi-Session.

### PHASES & ACTIONS

1. **DISCOVERY**:
   * **ARCH**: `delegate_to_agent` (codebase_investigator).
   * **MAP**: `list_directory` | `glob`.
   * **CODE**: `find_code` (Simple) > `dump_syntax_tree` > `find_code_by_rule` (Complex/Struct).
   * **LIBS**: `resolve-library-id` -> `query-docs`.
2. **SPEC**:
   * **DOCS**: `resolve-library-id` -> `query-docs`.
   * **PLAN**: **WRITE** detailed plan in `$TASKS/<id>.md` -> **UPDATE** `$CTX`.
3. **IMPL**:
   * **LOOP**: TDD -> `find_code` -> `replace`/`write`.
4. **VERIFY**:
   * **TEST**: Run project tests.
   * **DEBUG**: On Fail -> `resolve-library-id` -> `query-docs`.

### TRANSITIONS

* **START**: **TOUCH** `$TASKS/<id>.md` -> **UPDATE** `$CTX` ref `task_id`.
* **DONE**: Update `$TASKS` [x] -> **MV** to `$TASKS/completed/` -> **CLEAR** `$CTX` `task_id` -> Update `$MB/progress.md`.
* **SESSION_END**: **BACKFILL** learnings to `$TASKS` + `$MB` -> **STOP**.

## 2. OPERATIONAL LOOP

1. **SYNC**: **READ** `$CTX`. Check Phase.
2. **STRATEGY**:
   * **AMBIGUOUS**? -> **STOP** -> **ASK**.
   * **PHASE_MISMATCH**? -> **UPDATE** `$CTX` -> **GOTO** 1.
   * **EXECUTE**: Perform Phase Action (Sec 1).
3. **PERSIST**: Update `$CTX` (Current Focus).
