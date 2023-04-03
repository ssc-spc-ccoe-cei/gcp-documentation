# Testing Gatekeeper Constraints Using Gator CLI

The Gator CLI can be used to test Gatekeeper `ConstraintTemplates` and `Constraints` locally.

Details and latest information: <https://open-policy-agent.github.io/gatekeeper/website/docs/gator>

## Installation

You have a few options to install the Gator CLI:

- [Download the binary](https://github.com/open-policy-agent/gatekeeper/releases)

- To build from source:

    ```shell
    go get github.com/open-policy-agent/gatekeeper/cmd/gator
    ```

- Install with Homebrew:

    ```shell
    brew install gator
    ```

## The `gator test` subcommand

You can use the `gator test` subcommand to test manifest files against a set of policies.

You specify inputs using the `--filename` or short `-f` flag. Supported extensions are `.json`, `.yaml`, and `.yml`.

Example 1:

```shell
cat my-manifest.yaml | gator test --filename=template-and-constraints-folder/
```

Example 2:

```shell
gator test -f=my-manifest.yaml -f=templates-and-constraints-folder/
```

Example 3:

Run the following example scenario:

- Download the community-owned library of policies for the OPA Gatekeeper project.

    ```shell
    git clone https://github.com/open-policy-agent/gatekeeper-library.git
    ```

- Switch to the following folder:

    ```shell
    cd gatekeeper-library/library/general/httpsonly
    ```

- Run the tree command:

    ```shell
    tree
    .
    ├── kustomization.yaml
    ├── samples
    │   ├── ingress-https-only
    │   │   ├── constraint.yaml
    │   │   ├── example_allowed.yaml
    │   │   └── example_disallowed.yaml
    │   └── ingress-https-only-tls-optional
    │       ├── constraint.yaml
    │       ├── example_allowed.yaml
    │       └── example_disallowed.yaml
    ├── suite.yaml
    ├── sync.yaml
    └── template.yaml
    ```

- Test the `example_allowed.yaml` manifest against the constraintemplate and constraint.

    ```shell
    gator test -f=samples/ingress-https-only/example_allowed.yaml -f=template.yaml -f=samples/ingress-https-only/constraint.yaml
    ```

    This will not return any errors.

    **Tip:** Adding --output=json or --output=yaml will return a null value.

    ```shell
    gator test -f=samples/ingress-https-only/example_allowed.yaml -f=template.yaml -f=samples/ingress-https-only/constraint.yaml --output=json

    null
    ```

    ```shell
    gator test -f=samples/ingress-https-only/example_allowed.yaml -f=template.yaml -f=samples/ingress-https-only/constraint.yaml --output=yaml

    []
    ```

- Test the `example_disallowed.yaml` manifest against the constraintemplate and constraint.

    ```shell
    gator test -f=samples/ingress-https-only/example_disallowed.yaml -f=template.yaml -f=samples/ingress-https-only/constraint.yaml
    ```

    This will return the following error:

    ```shell
    Message: "Ingress should be https. tls configuration and allow-http=false annotation are required for ingress-demo-disallowed"
    ```

### Exit Codes

`gator test` will return the following information:

- A `0` exit status when the objects, Templates, and Constraints are successfully ingested, no errors occur during evaluation, and no violations are found.

- A `1` exit status code will be generated for policy violations. The violation will be printed to `stdout`.

You can send the output to `yaml` or `json` format:

```shell
gator test --filename=manifests-and-policies/ --output=json
```

```shell
gator test --filename=manifests-and-policies/ --output=yaml
```

## The `gator verify` subcommand

### Test Suites

`gator verify` organizes tests into three levels:

| Level | Description |
| -------- | --------- |
| Suites | A Suite is a file which defines Tests |
| Tests  | A Test declares a ConstraintTemplate, a Constraint, and Cases to test the Constraint  |
| Cases  | A Case defines an object to validate and whether the object is expected to pass validation  |

### Suites

A valid Suite file declares the following:

```shell
kind: Suite
apiVersion: test.gatekeeper.sh/v1alpha1
```

- `gator verify` will silently ignore files which do not declare these.

- A Suite may declare multiple Tests, each containing different Templates and Constraints. Each Test in a Suite is independent.

### Tests

Each Suite contains a list of Tests under the `tests` field.

### Cases

Each Test contains a list of Cases under the `cases` field.

A Case must specify assertions and whether it expects violations. The simplest way to declare this is:

The Case expects at least one violation:

```shell
assertions:
- violations: yes
```

The Case expects no violations:

```shell
assertions:
- violations: no
```

**For further information visit:**

<https://open-policy-agent.github.io/gatekeeper/website/docs/gator#cases>

### Usage

To run a specific suite:

```shell
gator verify suite.yaml
```

To run all suites in the current directory and all child directories recursively

```shell
gator verify ./...
```

To only run tests whose full names contain a match for a regular expression, use the run flag:

```shell
gator verify path/to/suites/... --run "disallowed"
```

## Creating a Test Suite

The following example will showcase the test suite that was built to evaluate the GCP project resource constraints.

Gatekeeper policies are stored in the `gcp-blueprints-catalog`.

The test suite, constraint, constrainttemplate, and tests can be found [here](https://dev.azure.com/gc-cpa/iac-gcp/_git/gcp-blueprints-catalog?path=/gatekeeper-policies/naming-rules/project).

The following steps can be used to create a test suite:

- Create a file called `suite.yaml`.

  - The following elements are important and define most of the suite file.
    - kind
    - apiVersion
    - tests
    - cases
    - violations
  - Set a name for the test suite under metadata - name.
  - This example only contains two test cases: one for `allowed` values and the other for `not allowed` values.
  - The `template.yaml` (ConstraintTemplate) and `constraint.yaml` (constraint) files should be in the same folder as the `suite.yaml` file.
    - The resource manifests contained in the `tests/` folder are evaluated against the constraints set in the `template.yaml` and `constraint.yaml` files.
  - Tests are manifest files containing example projects and are placed under `tests/`.
    - The `tests/project_allowed.yaml` manifest contains a project resource with a valid name while the `tests/project_not_allowed.yaml` file contains a project resource with an invalid name.
  - The `allowed` test case expects `no` violations while the `not_allowed` case expects one or more.
    - Violations that specifies a `yes` must at least contain one violation that matches the assertion.
    - Violations with a `no` must not contain any violations that matches the assertion.
    - If set to a non-negative integer, then exactly that many violations must match. Defaults to "yes".

    ```yaml
    kind: Suite
    apiVersion: test.gatekeeper.sh/v1alpha1
    metadata:
      name: namingpolicyproject
      annotations:
        config.kubernetes.io/local-config: 'true'
    tests:
    - name: namingpolicyprojecttests
      template: template.yaml
      constraint: constraint.yaml
      cases:
      # Allowed
      - name: allowed
        object: tests/project_allowed.yaml
        assertions:
        - violations: 'no'
      # Not allowed
      - name: notallowed
        object: tests/project_not_allowed.yaml
        assertions:
        - violations: 'yes'
    ```

- Create a folder called `tests` and place resource manifest files containing the gcp resource(s) being evaluated.
- Your folder structure should be similar to this:

    ```shell
    .
    ├── Kptfile
    ├── README.md
    ├── constraint.yaml
    ├── setters.yaml
    ├── suite.yaml
    ├── template.yaml
    └── tests
        ├── project_allowed.yaml
        └── project_not_allowed.yaml
    ```

- This example only expects one valid (allowed) value and one bad (not_allowed) value. The `tests/project_allowed.yaml` manifest contains a project resource with a valid name while the `tests/project_not_allowed.yaml` file contains a project resource with an invalid name.

    project_allowed.yaml

    ```yaml
    apiVersion: resourcemanager.cnrm.cloud.google.com/v1beta1
    kind: Project
    metadata:
      name: aadmu-pe-test-project
      annotations:
        config.kubernetes.io/local-config: 'true'
        cnrm.cloud.google.com/blueprint: kpt-pkg-fn
    spec:
      name: aadmu-pe-test-project
    ```

    project_not_allowed.yaml

    ```yaml
    apiVersion: resourcemanager.cnrm.cloud.google.com/v1beta1
    kind: Project
    metadata:
      name: zzxyq-pe-test-projectA
      annotations:
        config.kubernetes.io/local-config: 'true'
        cnrm.cloud.google.com/blueprint: 'kpt-pkg'
    spec:
      name: zzxyq-pe-test-projectA
    ```

- Test using `gator verify suite.yaml`

    **Note:** The output below as been edited for clarity.

    ```shell
    $ pwd
    gcp-blueprints-catalog/gatekeeper-policies/naming-rules/project
    $ gator verify suite.  yaml
    ok      gcp-blueprints-catalog/gatekeeper-policies/naming-rules/project/suite.yaml       0.034s
    PASS
    ```
