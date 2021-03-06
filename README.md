# Get chaged Kustomize overlays
When managing multiple applications across multiple clusters in a [GitOps approach with kustomize](https://github.com/operate-first/apps) there are a lot of overlays to test.

Kustomize doesn't even provide log output of it's transformations,
this tool recursively calculates the dependencies of a given overlay and compares them with the files changed by a merge request, returning only those kustomize overlays that were affected by a merge request.

> This tool does not check the validity of your kustomizations beyond resolving the paths of the dependencies,
> for proper linting use `oc apply --dry-run=server -f` or [kubeconform](https://github.com/yannh/kubeconform).

# Usage
```
usage: main.py [-h] --changed-files CHANGED_FILES [CHANGED_FILES ...] --base-overlays BASE_OVERLAYS [BASE_OVERLAYS ...] [-o OUTPUT_FILE]

optional arguments:
  -h, --help            show this help message and exit
  --changed-files CHANGED_FILES [CHANGED_FILES ...]
                        The files that were changed
  --base-overlays BASE_OVERLAYS [BASE_OVERLAYS ...]
                        The base overlays potentially affected
  -o OUTPUT_FILE, --output-file OUTPUT_FILE
                        The output file path
```

## Example
### Manually
```bash
python -m get-changed-kustomize-overlays --changed-files folder1/file1.yaml folder2/file2.yaml --base-overlays folder1/kustomization.yaml folder2/kustomization.yaml 
```

### GitLab CI
```bash
OVERLAYS=$(find . -regex '.*/overlays/[a-z]*/kustomization.yaml' | tr '\n' ' ')
CHANGED_FILES=$(git diff --name-only $CI_MERGE_REQUEST_TARGET_BRANCH_SHA $CI_MERGE_REQUEST_SOURCE_BRANCH_SHA | tr '\n' ' ')
python -m get-changed-kustomize-overlays --changed-files $(echo $CHANGED_FILES) --base-overlays $(echo $OVERLAYS)
```

# External dependencies
It is not possible to check for changes in external dependencies in CI pipelines that are triggered by merge requests, 
you should use tags or SHAs to signify changes in external dependencies.
```
 ---
 apiVersion: kustomize.config.k8s.io/v1beta1
 kind: Kustomization

 resources:
 - https://github.com/ORGANIZATION/REPOSITORY.git//PATH_INSIDE_REPO?ref=<TAG/SHA>
 ```



