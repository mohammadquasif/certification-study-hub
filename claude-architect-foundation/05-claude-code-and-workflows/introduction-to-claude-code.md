# Introduction to Claude Code — CLI, Tools & Capabilities

> **Exam:** Claude Architect Foundation
> **Chapter:** 05 — Claude Code & Developer Workflows
> **Difficulty:** Beginner
> **Last Updated:** 2026-06

---

## 📖 What Is This Topic About?

**Claude Code** is Anthropic's agentic coding tool that runs in your terminal. Unlike a chat interface, Claude Code can read and write files, execute shell commands, search codebases, and run tests — all as part of completing a coding task. Understanding what Claude Code can do, and how to use it effectively, is the final domain of this exam.

---

## 🧠 Key Concepts

- **Claude Code** — An agentic coding CLI (command line interface) that gives Claude access to file system, shell, and search tools to complete development tasks autonomously
- **Read tool** — Claude reads file contents; used to understand code before making changes
- **Write tool** — Claude creates or overwrites files with new content
- **Edit tool** — Claude makes targeted changes to specific sections of existing files (preferred over Write for surgical edits)
- **Grep tool** — Claude searches file contents using regular expressions; returns matching lines with context
- **Glob tool** — Claude finds files matching a pattern (e.g., all `*.test.js` files) without reading their contents
- **Bash / Shell tool** — Claude runs terminal commands (tests, linters, build scripts, git operations)
- **Plan Mode** — A mode where Claude reasons and plans before making any file changes

---

## 📘 Detailed Explanation

### What Claude Code Is — and Is Not

| Aspect | Claude Code | Claude (chat/API) |
|---|---|---|
| **Interface** | Terminal CLI | Chat UI / API |
| **File access** | Reads, writes, edits files | No file system access |
| **Command execution** | Runs shell commands (tests, git, npm) | No command execution |
| **Code search** | Grep, Glob across codebases | No search tools |
| **Memory** | Session-based (can be resumed/forked) | Stateless by default |
| **Use case** | Coding tasks, investigation, refactoring | Explanations, Q&A, generation |

---

### The Core File Tools — When to Use Each

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CLAUDE CODE FILE TOOLS                           │
│                                                                     │
│  READ — Understanding before acting                                 │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Use when: You need to understand code before changing it     │  │
│  │ Read specific files, config files, test files                │  │
│  │ Claude reads the content; context grows with each Read       │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  GREP — Finding patterns in code                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Use when: You know what text/pattern to search for           │  │
│  │ "Find all files that import AuthService"                     │  │
│  │ Returns matching lines with line numbers — efficient         │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  GLOB — Finding files by pattern                                    │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Use when: You need to find files by name/extension pattern   │  │
│  │ "Find all test files" → *.test.ts, *.spec.py                │  │
│  │ Returns file paths only; doesn't read content               │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  EDIT — Targeted surgical changes                                   │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Use when: Changing specific sections of existing files       │  │
│  │ Requires unique matching text to anchor the edit             │  │
│  │ Preferred over Write for existing files (safer)             │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  WRITE — Creating or replacing files                                │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │ Use when: Creating new files OR when Edit cannot find        │  │
│  │ unique anchor text                                           │  │
│  │ Overwrites entire file — use carefully                      │  │
│  └──────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

### When Grep Beats a Custom Shell Command

Claude Code has access to both Grep (a dedicated search tool) and Bash (shell commands). For searching, always prefer Grep:

```
SITUATION: Find all usages of eval() in a large codebase

❌ BASH APPROACH:
Claude could run: find . -name "*.py" -exec grep -n "eval(" {} \;
→ Works, but: Claude must write the command correctly, handle output parsing,
  and manage errors manually

✅ GREP TOOL:
Claude calls Grep with pattern "eval(" across the project directory
→ Returns structured results: file path, line number, matched line
→ No shell command needed, no parsing required, consistent format

RULE: Use specialised tools (Grep, Glob) for their specific purpose.
      Use Bash only when no specialised tool exists for the task.
```

---

### Plan Mode — Think Before You Code

For large, complex changes (architectural refactoring, multi-file changes, unfamiliar codebases), Claude Code should enter **Plan Mode** first:

```
PLAN MODE WORKFLOW:

Step 1: READ and explore relevant files
Step 2: GREP to understand imports, dependencies, usage patterns  
Step 3: REASON about the best approach and potential impact
Step 4: DOCUMENT the plan (what files to change, in what order, why)
        → User reviews the plan
Step 5: EXECUTE the plan (make actual file changes)

WITHOUT PLAN MODE (jumping straight to edits):
→ Risk of breaking dependent code
→ Risk of incomplete changes
→ Difficult to undo if approach was wrong

Example: Refactoring a monolith into microservices
         → Always plan first; understand all dependencies before changing anything
```

---

## 🇮🇳 Indian Real-Life Example

**Think of Claude Code like a senior software engineer at a TCS project:**

When a new requirement comes in, the senior engineer doesn't immediately start typing code. They:
1. **Read** the relevant existing code first (understand before touching)
2. **Search** for all places that call the function they're about to change
3. **Plan** the approach before making any changes
4. **Edit** specific sections — not rewrite the entire file
5. **Test** by running the test suite to verify nothing broke

Claude Code works exactly this way — read, search, plan, edit, test — before declaring the task complete.

---

## 🔑 Exam-Focused Points

- ✅ Claude Code = agentic CLI with file tools (Read, Write, Edit, Grep, Glob) + Bash
- ✅ **Grep** is the right tool for searching file contents; prefer it over custom shell commands
- ✅ **Glob** finds files by name pattern; does not read content
- ✅ **Edit** is preferred for targeted changes; **Write** for new files or when Edit can't find unique anchor
- ✅ **Plan Mode** is essential for large architectural changes — explore and plan before making changes
- ✅ Claude Code has session memory — sessions can be resumed, continued, or forked

---

## 🧩 Scenario-Based Thinking

**Scenario:** An engineer asks Claude Code to find all occurrences of an unsafe function `eval()` across a large Python codebase with hundreds of files. What exploration approach is most efficient?

- A) Have Claude read every file in the codebase until it finds eval() usages
- B) Use Grep to search for the pattern "eval(" across all Python files
- C) Use Glob to find all *.py files, then read each one to find eval() manually
- D) Run a Bash command to search for eval() using find + grep

**Answer:** B

**Explanation:** Grep is the purpose-built tool for this task. It searches file contents for a pattern and returns matching lines with file paths and line numbers — directly useful for an investigation. Reading every file (A) is inefficient. Glob + manual read (C) is Grep without the search capability. Bash commands (D) can do the same but Grep's structured output is more reliable and doesn't require writing a shell command.

---

## 💡 Memory Tricks

**Grep = Google Search for Code:** You know what you're looking for — type the search term, get results immediately. Don't manually browse every file.

**Edit vs Write:** Edit is like editing a specific paragraph in a Word document. Write is like deleting the whole document and typing a new one. Use Edit when possible — it's surgical and safer.

---

## ❓ Chapter Practice Questions

**Q1.** A developer asks Claude Code to understand how a caching module works before making changes. The module spans many files with complex inheritance hierarchies. What exploration approach will most efficiently build accurate understanding?

- A) Read every file in the codebase from top to bottom
- B) Use Grep to search for the word "cache" across all files
- C) Use Glob to find all files containing "cache" in the filename, then analyse imports and class hierarchies to identify the base cache class, read that file, and trace specific implementations
- D) Run the application and observe cache behaviour through logging

**Answer:** C

**Explanation:** Architecture-first exploration is more efficient than reading everything. Starting with file patterns (Glob), identifying the base class, reading the interface, then tracing specific implementations builds understanding bottom-up without reading irrelevant files. Searching for "cache" everywhere (B) returns too many unrelated results; reading everything (A) is prohibitively slow.

---

**Q2.** Claude Code is inserting a new helper function into the middle of a 200-line utility module. The Edit tool fails because the file has many repetitive docstrings and variable names that prevent finding unique anchor text. What is the most reliable alternative approach?

- A) Use Bash to append the function to the end of the file using a heredoc
- B) Read the complete file content, add the function at the correct location in memory, then Write the updated file
- C) Use Edit with an extremely long old_string capturing 50+ lines to guarantee uniqueness
- D) Use Edit's replace_all parameter with a common pattern

**Answer:** B

**Explanation:** When Edit cannot find a unique anchor, the most reliable fallback is Read (load the full file) → modify in memory → Write (overwrite with the updated version). This is safe because Claude sees the complete file content and can make the insertion at exactly the right location. Appending to the end (A) puts the function in the wrong place.

---

**Q3.** When is Plan Mode most essential in a Claude Code workflow?

- A) For all tasks, regardless of complexity
- B) For large architectural changes involving many files and significant design decisions — where incorrect assumptions would be costly to reverse
- C) Only when the user explicitly requests a plan
- D) For tasks that involve reading more than 5 files

**Answer:** B

**Explanation:** Plan Mode is most valuable for large, complex, or potentially irreversible changes. For a simple single-file bug fix or adding a comment, Plan Mode is unnecessary overhead. For restructuring a monolith, refactoring across 30 files, or making architectural decisions, exploring and planning first prevents costly mistakes.

---

## 📌 Quick Revision Summary

- Claude Code = agentic CLI: Read, Write, Edit, Grep, Glob, Bash tools + session memory
- Grep: search file contents for patterns — prefer over custom Bash search commands
- Glob: find files by name/extension pattern — doesn't read content
- Edit: targeted changes to specific sections (requires unique anchor text); Write: entire file
- Plan Mode: explore + plan before making changes for large/complex/architectural tasks
- Session memory: sessions can be resumed by name or ID, or forked for alternative explorations

---

## 📎 References

- [Anthropic Documentation — Claude Code](https://docs.anthropic.com/en/docs/claude-code)

---

*Notes by certification-study-hub. Chapter 05 — Introduction to Claude Code.*
