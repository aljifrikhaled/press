# Implementation Guide for Core Functions

This document serves as an implementation roadmap for developers looking to understand or modify the core business logic of Press.

## 1. Document Method Architecture (`run_doc_method`)
Almost all actions initiated from the Dashboard use `frappeRequest` pointing to `press.api.client.run_doc_method`.

To implement a new feature:
1.  Add a python method to the relevant DocType controller (e.g., `def my_new_feature(self):`).
2.  In the Vue Dashboard, use `frappeRequest` passing the DocType (`dt`), Document Name (`dn`), and the new method name (`method`).
3.  Ensure the method handles permissions correctly (Frappe will check if the user has read/write access to the document automatically).

## 2. Managing Site Lifecycles
The `Site` DocType (`press/press/doctype/site/site.py`) is the largest controller.

*   **State Changes:** Do not manually change `site.status = 'Active'`. The status is managed asynchronously. If you need to manipulate a site, you must dispatch an `Agent Job`.
*   **Dispatching Jobs:** Use the `Agent` class wrapper (`press/agent.py`).
    *   Example: `Agent(site.server).clear_site_cache(site)`
*   **Handling Async Results:**
    *   When the Agent finishes the job, `poll_pending_jobs` detects the completion.
    *   It triggers `process_job_updates` in `agent_job.py`.
    *   You must implement a handler function (e.g., `process_my_new_job_update(job)`) and route the specific `job_type` to your handler within `process_job_updates` to finalize the state change in the Press database.

## 3. Working with Site Actions
For complex, multi-step workflows (like moving a site to a different cluster), Press uses the `Site Action` DocType (`press/press/doctype/site_action/site_action.py`).

*   A `Site Action` allows defining sequential steps (e.g., 1. Archive Site -> 2. Provision New VM -> 3. Restore Site).
*   It implements a state machine (`StepStatus`) to track progress, pause on failures, and resume execution. Use `Site Action` instead of massive nested background jobs for multi-server operations.

## 4. Scaling Logic
When adjusting how Press allocates resources:
*   Look at `press/press/doctype/bench/bench.py` -> `allocate_workers()`.
*   Press calculates the `workload` based on the sum of `cpu_time_per_day` of all sites on the bench.
*   It calculates `usable_ram` on the server and distributes Gunicorn/RQ workers proportionally based on `MIN_GUNICORN_WORKERS` and `MAX_GUNICORN_WORKERS` constants.