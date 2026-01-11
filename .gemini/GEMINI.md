# SYSTEM: GEMINI || MODE: AGENT_CLI

## 0. BOOT_SEQUENCE

**VARS**: `$MB=./.memory-bank`, `$TASKS=./.tasks`, `$CTX=$MB/activeContext.md`
**DEV_MODE**: GREENFIELD (Non-BW Compat).

1. **INIT**: IF ABSENT `$MB`: **MKDIR** -> **CP** `.gemini/templates/*` `$MB/` -> **MV** `$MB/*.template.md` `$MB/*.md`. IF ABSENT `$TASKS`: **MKDIR**.
2. **LOAD**: **READ** `$MB/projectBrief.md` `$MB/progress.md` `$MB/systemContext.md` `$CTX`.
3. **ALIGN**: IF `$CTX` refs `task_id` -> **READ** `$TASKS/<task_id>.md`.

## 1. TASK_PROTOCOL (State Machine)

**HIER**: `$MB/progress.md` (Epics) > `$TASKS/*.md` (Stories) > `$CTX` (Session State).
**GUARD**: Large Task? -> **REQUIRE** Detailed Plan + Multi-Session.

### PHASES & ACTIONS

1. **DISCOVERY**: Context & Feasibility.
   * **ARCH**: `delegate_to_agent` (Root Cause/Sys Design/Refactor).
   * **MAP**: `list_directory` (Tree) / `glob` (File Loc).
   * **CODE**: `find_code` (Symbol) > `dump_syntax_tree`/`test_match_code_rule` (Refine) > `find_code_by_rule` (Struct).
   * **LIBS**: `resolve-library-id` -> `query-docs` (Validate usage).
2. **SPEC**: Requirements & Docs.
   * **DOCS**: `resolve-library-id` -> `query-docs` (Patterns/Examples).
   * **PLAN**: **WRITE** detailed plan in `$TASKS/<id>.md` -> **UPDATE** `$CTX` ref.
3. **IMPL**: Execution.
   * **LOOP**: TDD -> `find_code` (Locate) -> `replace`/`write` (Edit).
4. **VERIFY**: Quality Assurance.
   * **TEST**: Run project tests. **NEVER** leave broken builds.
   * **DEBUG**: On Fail -> `resolve-library-id` -> `query-docs` (Verify assumptions).

### TRANSITIONS

* **START**: **TOUCH** `$TASKS/<id>.md` -> **UPDATE** `$CTX` ref `task_id`.
* **DONE**: Update `$TASKS` [x] -> **MV** to `$TASKS/completed/` -> **CLEAR** `$CTX` `task_id` -> Update `$MB/progress.md`.
* **SESSION_END**: **BACKFILL** learnings to `$TASKS` + `$MB` -> **STOP**.

## 2. OPERATIONAL LOOP

1. **SYNC**: **READ** `$CTX`. Check Phase.
2. **STRATEGY**:
   * **AMBIGUOUS**? -> **STOP** -> **ASK** User.
   * **PHASE_MISMATCH**? -> **UPDATE** `$CTX` -> **GOTO** 1.
   * **EXECUTE**: Perform Phase Action (Sec 1).
3. **PERSIST**: Update `$CTX` (Current Focus). Chat=Ephemeral, Files=Persistent.
