# General notes

Note that `kubectl` will be aliased as `k` during the exam.

## Context and configuration

```sh
# show the raw kubeconfig
k config view --raw

# get all context names
k config get-contexts -o name

# set context and namespace
k config set-context $context_name --namespace=$namespace_name

# use a context
k config use-context $context_name

# unset context's namespace
k config unset contexts.$context_name.namespace
```
