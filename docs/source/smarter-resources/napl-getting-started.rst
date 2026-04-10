.. _napl-getting-started:

Getting Started: Claude Code + Smarter at NAPL
===============================================

.. rubric:: Northern Aurora Power & Light — Custom Programming Area

This tutorial gets you up and running as a **virtual CoPilot coding pair** —
using Claude Code as your AI coding assistant, routed through NAPL's on-premise
Smarter instance so that all traffic stays on corporate infrastructure.


Goal
----

We will configure Claude Code to route its LLM calls through the NAPL Smarter
API endpoint, then verify the integration by generating a small Python utility
function with Claude Code's assistance — entirely from the command line,
without sending data outside NAPL's network boundary.


Prerequisites
-------------

You are assumed to be comfortable with all of the following:

- Command-line usage (bash, PowerShell, or zsh).
- Python 3.10+ and ``pip``.
- Your IDE of choice (VS Code is recommended — a
  `Smarter VS Code Extension <https://marketplace.visualstudio.com/items?itemName=querium.smarter-manifest>`__
  is available).
- Git and basic GitHub workflow (clone, commit, push, pull request).
- REST APIs — you understand what a bearer token is and how
  ``Authorization: Token <value>`` headers work.

You have already been given a **Smarter account** by NAPL IT and you know how
to log in to the web console.  If you have not received credentials, open a
ticket with the IT Service Desk before continuing.


Setup
-----

**1. Install Claude Code**

Claude Code is Anthropic's official CLI for agentic coding.  Install it
globally via npm:

.. code-block:: console

   npm install -g @anthropic-ai/claude-code

Verify the installation:

.. code-block:: console

   claude --version

You should see a version string such as ``1.x.x``.

**2. Retrieve your Smarter API token**

Log in to the NAPL Smarter web console at the URL provided by IT.  Navigate to
**Account → API Tokens** and copy your personal token.  Keep it secret — it
carries the same permissions as your Smarter account.

**3. Identify the NAPL Smarter base URL and model**

Your infrastructure team will have communicated the internal hostname.  For
this tutorial we use the placeholder ``https://smarter.napl.internal``.  The
Anthropic Provider and ``claude-sonnet-4-6`` model have already been
configured by the platform team (see
:doc:`napl-add-llm-provider`).

**4. Set environment variables**

Claude Code reads its configuration from environment variables.  Add the
following to your shell profile (``~/.bashrc``, ``~/.zshrc``, or Windows
``$PROFILE``):

.. code-block:: bash

   # Route Claude Code through NAPL Smarter
   export ANTHROPIC_BASE_URL="https://smarter.napl.internal/api/v1/anthropic"
   export ANTHROPIC_API_KEY="<your-smarter-api-token>"
   export CLAUDE_MODEL="claude-sonnet-4-6"

Reload your profile:

.. code-block:: console

   source ~/.bashrc   # or: . $PROFILE  on PowerShell


Concept Overview
----------------

**Smarter as an LLM Proxy**

Smarter sits between your developer tools and the upstream LLM provider.
When Claude Code makes a request, it sends it to Smarter's
``/api/v1/anthropic`` endpoint using your Smarter token.  Smarter validates
the request, applies account-level rate limits and cost accounting, logs the
transaction in the Smarter Journal, and then forwards the call to Anthropic on
behalf of your account using the centrally managed ``ANTHROPIC_API_KEY``
secret.  The response is proxied back to Claude Code transparently.

.. code-block:: text

   Claude Code CLI
       │  ANTHROPIC_BASE_URL → https://smarter.napl.internal/api/v1/anthropic
       │  Authorization: Token <smarter-token>
       ▼
   NAPL Smarter (on-premise)
       │  Provider: Anthropic  │  Model: claude-sonnet-4-6
       │  Cost accounting / logging / rate limiting
       ▼
   Anthropic API  →  claude-sonnet-4-6

**Why this matters for NAPL**

- **Data sovereignty** — prompts and completions never leave the NAPL network
  perimeter in plaintext.
- **Cost accountability** — token consumption is tracked per account and cost
  code, giving IT accurate chargeback data.
- **Centralised key management** — developers never hold the production
  Anthropic key; Smarter's Secret store handles rotation transparently.
- **Audit trail** — every LLM call is logged in the Smarter Journal with
  timestamp, user, model, token count, and latency.

**Claude Code's CoPilot mode**

Claude Code ships with an interactive REPL and a ``--print`` flag for
non-interactive use.  In CoPilot mode (``claude`` with no arguments) it reads
your project context, understands your codebase, and can write, refactor, or
explain code in a conversational loop.  All of this works unchanged once
``ANTHROPIC_BASE_URL`` is pointed at Smarter.


Step-by-Step
------------

**Step 1 — Smoke-test the connection**

Before opening a project, confirm that Claude Code can reach Smarter:

.. code-block:: console

   claude --print "Reply with the single word: connected"

Expected output:

.. code-block:: text

   connected

If you see an authentication error, re-check ``ANTHROPIC_API_KEY`` and confirm
the Smarter URL is reachable from your machine:

.. code-block:: console

   curl -I https://smarter.napl.internal/api/v1/anthropic

**Step 2 — Open a project**

Navigate to any existing Python project (or create a scratch directory):

.. code-block:: console

   mkdir ~/napl-copilot-demo && cd ~/napl-copilot-demo
   git init

**Step 3 — Start an interactive CoPilot session**

.. code-block:: console

   claude

Claude Code scans the current directory, then presents its interactive prompt.
You will see something like:

.. code-block:: text

   ╭─────────────────────────────────────╮
   │ ✻ Welcome to Claude Code!           │
   │   Model: claude-sonnet-4-6          │
   │   Base URL: smarter.napl.internal   │
   ╰─────────────────────────────────────╯
   >

**Step 4 — Ask Claude Code to write a function**

At the prompt, type:

.. code-block:: text

   Write a Python function called `parse_meter_reading` that accepts a raw
   string like "12345.67 kWh" and returns a tuple of (float, str) representing
   the numeric value and the unit.  Include a docstring and two doctest
   examples.

Claude Code will propose a new file.  Accept the change when prompted.

**Step 5 — Iterate**

Ask a follow-up question in the same session:

.. code-block:: text

   Add a ValueError if the unit is not one of: kWh, MWh, GWh.

Review the diff Claude Code proposes before accepting each change.

**Step 6 — Exit and commit**

Type ``/exit`` or press ``Ctrl+C`` to end the session, then commit your work:

.. code-block:: console

   git add .
   git commit -m "feat: add parse_meter_reading utility via Claude Code CoPilot"


Proof of Concept
----------------

After completing the steps above, your project should contain a file similar
to the following:

.. code-block:: python
   :caption: parse_meter_reading.py (Claude Code generated)

   def parse_meter_reading(raw: str) -> tuple[float, str]:
       """Parse a raw meter reading string into a numeric value and unit.

       Args:
           raw: A string in the format "<number> <unit>", e.g. "12345.67 kWh".

       Returns:
           A tuple of (value: float, unit: str).

       Raises:
           ValueError: If the unit is not one of kWh, MWh, or GWh.

       Examples:
           >>> parse_meter_reading("12345.67 kWh")
           (12345.67, 'kWh')
           >>> parse_meter_reading("9.1 MWh")
           (9.1, 'MWh')
       """
       VALID_UNITS = {"kWh", "MWh", "GWh"}
       parts = raw.strip().split()
       if len(parts) != 2:
           raise ValueError(f"Unexpected format: {raw!r}")
       value, unit = float(parts[0]), parts[1]
       if unit not in VALID_UNITS:
           raise ValueError(f"Invalid unit {unit!r}. Must be one of {VALID_UNITS}.")
       return value, unit

Run the doctests to confirm correctness:

.. code-block:: console

   python -m doctest parse_meter_reading.py -v

All tests should pass.  You have successfully used Claude Code via NAPL's
Smarter platform to generate production-quality, tested Python code without
leaving your terminal or exposing data outside the corporate network.


Troubleshooting
---------------

.. list-table::
   :header-rows: 1
   :widths: 35 65

   * - Symptom
     - Resolution
   * - ``AuthenticationError`` on first ``claude`` command
     - Confirm ``ANTHROPIC_API_KEY`` is your **Smarter** token (not a raw
       Anthropic key). Smarter uses this field as its bearer token.
   * - ``ConnectionRefusedError`` or timeout reaching Smarter
     - Ensure you are on the NAPL corporate network or VPN.  Verify the URL
       with ``curl -I https://smarter.napl.internal/api/v1/anthropic``.
   * - Model not found error
     - The ``claude-sonnet-4-6`` Provider Model may not be active yet.
       Contact the Smarter platform administrator or check
       :doc:`napl-add-llm-provider`.
   * - Claude Code shows wrong model in header
     - Confirm ``CLAUDE_MODEL`` is exported in your shell profile and that you
       reloaded the profile (``source ~/.bashrc``).
   * - Responses are much slower than expected
     - Smarter routes calls through your on-premise infrastructure.  If
       latency is consistently > 30 s, open an IT ticket referencing the
       Smarter service — it may indicate a Celery worker bottleneck.
   * - Cost codes not appearing in IT charge-back reports
     - Ask your account administrator to confirm the correct cost code is
       associated with your Smarter account in **Dashboard → Account →
       Cost Codes**.

.. seealso::

   - :doc:`Adding an LLM Provider <napl-add-llm-provider>`
   - :doc:`Smarter Platform Quick Start <../smarter-platform/quick-start>`
   - `Claude Code Documentation <https://docs.anthropic.com/en/docs/claude-code>`__
   - `Anthropic API Reference <https://docs.anthropic.com/en/api/>`__
