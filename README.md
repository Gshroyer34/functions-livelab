# From Code to Cloud in Minutes with OCI Functions

Cloud-tested authoring package, September 17, 2026. Eleven infrastructure checks,
ten local tests, all four event-driven cloud exercises, and Task 2's actual
Cloud Shell checks/repackaging pass.

## Agreed scope

- Beginner audience; 30-45 minutes of lab activity, targeting 35-40 minutes.
- Object Storage upload -> OCI Events -> one Python function -> two CSV reports.
- A supplied prompt for AI-generated processing code, plus our reference module.
- Threshold change and invalid-row/log diagnosis are optional extensions.
- Chicago (`us-chicago-1`), using the `LiveLab` compartment.
- Supporting infrastructure is prepared for learners. Convert the proven setup
  to Terraform and integrate with the green-button platform in a later phase.


## Package layout

```text
functions-livelab/
  tutorial/
    index.md                  Learner guide; author-review draft
    img/                      Actual console screenshots and capture checklist
    files/
      ai-prompt.txt
      inventory-lab-source.zip
      inventory-run1.csv
      inventory-run2.csv
      inventory-bad.csv
      inventory-fixed.csv
      verify_inventory.py
      expected/               Expected reports for all four runs
      function/
        inventory.py          Standard-library processing module
        func.py               OCI event/Object Storage/logging adapter
        requirements.txt
        func.yaml             Standard Fn deployment metadata
  author/
    provision.ps1             Repeatable CLI setup for this authoring tenancy
    build_assets.py           Regenerates fixtures, expected reports, source ZIP
    test_adapter.py           Local adapter checks using fake Object Storage
    package_function.py       Compact Linux dependency archive builder
    test_packaged_sdk.py       Isolated SDK authentication/client check
    check_environment.ps1     Read-only infrastructure checks
    connect_event.ps1         Create the lab's scoped event rule
    test_cloud.ps1            Real upload-to-report acceptance checks
    INITIALIZATION.md         Green-button/Terraform setup specification
    VALIDATION.md             Evidence, known issues, and outstanding checks
    state/                    Local resource inventory; excluded from publication
```

Use `tutorial/` as the content package, following the supplied OSPA markdown
template. Keep author tooling, cloud resource OCIDs, and working logs outside
the published learner bundle.

## Local validation

From the workspace root, run:

```text
python functions-livelab/author/build_assets.py
python functions-livelab/tutorial/files/verify_inventory.py
python functions-livelab/author/test_adapter.py
```

The inventory checker has six tests, including four 20-row fixtures, boundaries,
normalization, header validation, quoted CSV fields, and continued processing
after rejected rows. The adapter suite has four tests covering expected outputs,
repeat delivery, ignored events, and bad-file processing. These are local checks;
they do not establish deployment, IAM, event delivery, or logging success in OCI.

| Input | Threshold | Restock rows | Rejected rows |
| --- | ---: | ---: | ---: |
| inventory-run1.csv | 10 | 5 | 0 |
| inventory-run2.csv | 20 | 11 | 0 |
| inventory-bad.csv | 20 | 10 | 1 |
| inventory-fixed.csv | 20 | 11 | 0 |

Quantities and records INV-001 through INV-011 follow the storyboard. The
remaining nine fixture rows are newly supplied test data with quantities >= 20.
The bad file changes only INV-007 from `12` to `twelve`; the fixed file restores
`12`. Counts exclude headers.

## Verified Cloud Shell learner commands

Task 2 uses the actual OCI Cloud Shell terminal, not a local Python installation.
The exact downloadable source ZIP was uploaded and extracted there on September
17, 2026. All six inventory tests passed on the default Python 3.9.25 and explicit
Python 3.12.14, including a Python 3.12 run with site packages disabled (`-S`).
The guide uses `python3.12` to match the deployed function's language version.

`python3.12 package_logic.py` created the custom ZIP; its entry list, integrity,
all dependency bytes, and four replacement source files were verified. Cloud
Shell's download dialog takes `inventory-lab/inventory-reporter-custom.zip`
relative to the home directory, regardless of the terminal's current directory.
Neither learner helper needs pip, OCI SDK, FDK, Docker, or cloud API calls.
See `author/VALIDATION.md` for the environment details and remaining IAM gate.

## Deployment path and validation gate

An application groups functions; it is not itself a code editor or deployment
method. Code Editor is a browser workspace. The established Fn workflow builds
an image and pushes it to OCI Container Registry before defining the function.
The code-only path accepts a ZIP and uses a managed runtime. Console direct
upload is verified to create an Active function in Chicago, with runtime
`python312.ol9`, handler `func.handler`, 256 MB, and a 60-second timeout.

The installed OCI CLI (3.89.3) does not expose code-only commands. Management API
`20260325` does expose archive metadata and asynchronous work requests; signed
requests are available through `oci raw-request`. See Oracle's
[management API change notice](https://docs.oracle.com/en-us/iaas/Content/servicechanges.htm).

The archive must contain a `function/` directory. The Python runtime supplies
FDK, but an actual invocation showed that it does NOT install `requirements.txt`.
`author/package_function.py` therefore bundles the OCI SDK and Linux/Python 3.12
dependencies with the source. It retains Object Storage and SDK shared-helper
dependencies (including DNS and Work Requests models), excluding unrelated
services so the ZIP fits the direct-upload path. Learners reuse dependencies when replacing
`inventory.py` with `package_logic.py`; they do not need Docker or an image push.

`inventory-reporter.zip` is the deployment artifact. `inventory-lab-source.zip`
is the outer teaching bundle and must not be uploaded as the function archive.
All four CSV exercises passed end-to-end, with exact expected report matches and
verified rejected-row logs. The threshold was restored to 10. See
[`author/VALIDATION.md`](author/VALIDATION.md) for evidence and packaging lessons,
and [`author/INITIALIZATION.md`](author/INITIALIZATION.md) for the green-button specification.

Author-only build (Linux x86-64/Python 3.12; not a learner step):

```text
docker run --rm --platform linux/amd64 --mount type=bind,source=<absolute-functions-livelab-directory>,target=/lab python:3.12-slim python /lab/author/package_function.py
```

This installs the pinned OCI SDK from PyPI, bundles compatible Linux libraries,
and tests the exact extracted artifact with synthetic resource-principal
credentials. No cloud calls or real credentials are used by that packaging
check. Dependencies are built for the x86 application, not Arm. Rebuild the outer
source bundle with `build_assets.py` after changing the deployment archive.

## Infrastructure and publication checks

`author/provision.ps1` creates named, tagged resources and records their OCIDs in
`author/state/resources.json`. It reuses only resources with the lab's management
tag and does not change or remove unrelated resources. IAM requests use the
tenancy's home region; the workload uses Chicago. The current script is an
authoring aid, not the finished green-button provisioning definition.

The design uses a private subnet, an Oracle Services Network gateway, no inbound
security-list rules, and HTTPS egress to Oracle services. Buckets remain private.
The incoming bucket emits events; the output bucket does not. A dynamic group
selects functions in LiveLab, with input-object read and output-object create/
overwrite permissions. There are no personal credentials in the function.

Before publishing:

- Verify code-only availability and the reference archive in the learner tenancy.
- Finish screenshot/publication review; the authoring cloud flow is verified.
- Repeat the verified Cloud Shell workflow under the final participant role.
- Run a novice timing test; adjust estimated times from observed results.
- Validate least-privilege learner permissions and teardown in a fresh sandbox.
- Confirm the final author/contact information and artifact hosting URL.

Resources are retained for the next authoring and screenshot session. This
package does not run cleanup automatically.
