.. _napl-add-llm-provider:

Adding an LLM Provider: Anthropic & Claude Code
================================================

This tutorial walks you through adding Anthropic as an LLM Provider in the
Smarter platform, and registering **claude-sonnet-4-6** as an active Provider
Model. Once registered, Smarter Chatbots and Prompts in your account can
reference this model by name.

.. note::

   Provider management was introduced in Smarter **v0.11.0**. Confirm your
   installation is at least this version before proceeding.

   Your Smarter account must have **Account Administrator** privileges to add
   and manage providers.

Prerequisites
-------------

- A Smarter account with Account Administrator access.
- An active `Anthropic API key <https://console.anthropic.com/settings/keys>`__
  (begins with ``sk-ant-``).
- The Smarter Secret for storing your API key has already been created (see
  :doc:`Account Management <../smarter-platform/account-management>`).

.. tip::

   If you have not yet stored your Anthropic API key as a Smarter Secret,
   navigate to **Dashboard → Secrets → Add Secret** and save it before
   continuing. Secrets are encrypted at rest and referenced by name — never
   pasted directly into Provider fields.


Step 1 — Open the Provider Dashboard
-------------------------------------

Log in to the Smarter web console and click **Providers** in the left-hand
navigation menu.  If this is the first provider you are adding, the list will
be empty.

Click **+ Add Provider**.


Step 2 — Fill in Provider Metadata
------------------------------------

Complete the form fields as shown below.  Fields marked with ``*`` are
required.

.. list-table:: Anthropic Provider Fields
   :header-rows: 1
   :widths: 30 20 50

   * - Field
     - Value
     - Notes
   * - **Name** \*
     - ``Anthropic``
     - Used as the internal identifier. Must be unique within your account.
   * - **Base URL** \*
     - ``https://api.anthropic.com``
     - Anthropic's production API root. Do not include a trailing slash.
   * - **Connectivity Test Path**
     - ``/v1/models``
     - The path Smarter appends to Base URL when running the API connectivity
       verification check.
   * - **API Key** \*
     - *(select your Anthropic Secret)*
     - Choose the Secret you created in the prerequisite step.
   * - **Website URL**
     - ``https://www.anthropic.com``
     -
   * - **Docs URL**
     - ``https://docs.anthropic.com``
     -
   * - **Contact Email**
     - ``support@anthropic.com``
     -
   * - **Support Email**
     - ``support@anthropic.com``
     -
   * - **Terms of Service URL**
     - ``https://www.anthropic.com/legal/aup``
     -
   * - **Privacy Policy URL**
     - ``https://www.anthropic.com/legal/privacy``
     -
   * - **Logo**
     - *(optional image upload)*
     - PNG or SVG of the Anthropic mark.

Click **Save**.  The Provider record is created with a status of
``Unverified``.


Step 3 — Accept the Terms of Service
--------------------------------------

After saving, Smarter displays a **Terms of Service** acceptance prompt.  Read
the linked Anthropic AUP, then click **Accept Terms of Service**.  Smarter
records the accepting user and timestamp in the ``tos_accepted_by`` and
``tos_accepted_at`` fields.

.. important::

   The provider will not advance past ``Unverified`` until the Terms of
   Service have been accepted.


Step 4 — Trigger Verification
-------------------------------

Click **Verify Provider** to start the automated verification workflow.
Smarter runs the following checks asynchronously:

.. list-table:: Provider Verification Checks
   :header-rows: 1
   :widths: 30 70

   * - Verification Type
     - What Smarter Checks
   * - ``api_connectivity``
     - HTTP ``GET`` to ``https://api.anthropic.com/v1/models`` using your
       stored API key. Expects a ``200 OK``.
   * - ``logo``
     - Provider logo is present and renderable.
   * - ``contact_email``
     - Contact email field is non-empty.
   * - ``support_email``
     - Support email field is non-empty.
   * - ``website_url``
     - Website URL is reachable.
   * - ``tos_url``
     - Terms of Service URL is reachable.
   * - ``privacy_policy_url``
     - Privacy Policy URL is reachable.
   * - ``tos_acceptance``
     - User has accepted the Terms of Service.
   * - ``production_api_key``
     - Environment variable ``ANTHROPIC_API_KEY`` is set (applies to
       production deployments managed by your infrastructure team).

When all checks pass, the provider status transitions to ``Verified`` and
``is_active`` is set to ``True``.


Step 5 — Add a Provider Model
-------------------------------

With the Provider verified, click **Add Model** within the Anthropic Provider
record.  Complete the Provider Model form:

.. list-table:: claude-sonnet-4-6 Model Fields
   :header-rows: 1
   :widths: 35 25 40

   * - Field
     - Value
     - Notes
   * - **Name** \*
     - ``claude-sonnet-4-6``
     - Exact model string used in Anthropic API requests.
   * - **Description**
     - ``Claude Sonnet 4.6 — balanced speed and intelligence``
     -
   * - **Max Completion Tokens** \*
     - ``8192``
     - Anthropic's default output cap for Sonnet-class models.
   * - **Temperature** \*
     - ``0.7``
     - Controls response creativity. Range 0.0 – 1.0.
   * - **Top P** \*
     - ``1.0``
     -
   * - **Is Default**
     - ✓ (checked)
     - Makes this the default model for new Chatbots referencing Anthropic.
   * - **Supports Streaming**
     - ✓
     - Claude supports SSE streaming.
   * - **Supports Tools**
     - ✓
     - Claude supports native tool/function calling.
   * - **Supports Text Input**
     - ✓
     -
   * - **Supports Image Input**
     - ✓
     - Claude is multimodal.
   * - **Supports Text Generation**
     - ✓
     -
   * - **Supports Code Interpreter**
     - ✓
     - Relevant for Claude Code integration.

Click **Save**.  Smarter will run Provider Model verification checks
(streaming, tools, text input) asynchronously.  Once complete, set
**Is Active** to ``True``.


Step 6 — Reference the Model in a Smarter Manifest
----------------------------------------------------

You can now reference ``Anthropic`` and ``claude-sonnet-4-6`` in any Smarter
Application Manifest (SAM) YAML file:

.. code-block:: yaml
   :caption: Example chatbot manifest referencing Claude

   apiVersion: smarter/v1
   kind: Chatbot
   metadata:
     name: my-napl-chatbot
     account: napl-custom-programming
   spec:
     provider: Anthropic
     model: claude-sonnet-4-6
     system_prompt: |
       You are a helpful coding assistant for the NAPL custom programming team.
     max_completion_tokens: 8192
     temperature: 0.7

Apply the manifest:

.. code-block:: console

   smarter apply -f my-napl-chatbot.yaml


Verification
------------

To confirm everything is wired correctly, call the Smarter REST API:

.. code-block:: console

   curl -s https://<your-smarter-host>/api/v1/providers/ \
        -H "Authorization: Token <your-smarter-token>" | \
        python3 -m json.tool

Look for an entry with ``"name": "Anthropic"`` and
``"status": "verified"``.  A ``200`` response with this payload confirms
the integration is live.


Troubleshooting
---------------

.. list-table::
   :header-rows: 1
   :widths: 35 65

   * - Symptom
     - Resolution
   * - ``api_connectivity`` check fails
     - Confirm ``ANTHROPIC_API_KEY`` is set in ``.env`` and the key begins
       with ``sk-ant-``. Rotate the key in the Anthropic console if expired.
   * - Status stays ``Verifying`` for > 5 minutes
     - Check the Celery worker logs: ``docker compose logs celery``. A stalled
       verification task usually means the worker is not running.
   * - ``production_api_key`` check fails in production
     - Ensure your infrastructure team set ``ANTHROPIC_API_KEY`` as an
       environment variable in the Kubernetes Helm values or the server's
       ``systemd`` environment file.
   * - Model not visible in Chatbot authoring UI
     - Confirm both ``Provider.is_active`` and ``ProviderModel.is_active``
       are ``True`` in the Django admin or via the Smarter dashboard.

.. seealso::

   - :doc:`Smarter Provider Reference <smarter-provider>`
   - :doc:`Smarter Quick Start <../smarter-platform/quick-start>`
   - `Anthropic API Documentation <https://docs.anthropic.com/en/api/>`__
