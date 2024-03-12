# Trusted POD

The `io.containerd.runc.v2` shim automatically groups based on the presence of [labels](https://github.com/containerd/containerd/blob/b30e0163ac36c1a193604e5eca031053d62019c5/runtime/v2/runc/manager/manager_linux.go#L54-L60). 

In practice, this means that containers launched by Kubernetes.

And that are part of the same Kubernetes pod, are handled by a single shim.

They are grouping on the `io.kubernetes.cri.sandbox-id` label set by the CRI plugin.
