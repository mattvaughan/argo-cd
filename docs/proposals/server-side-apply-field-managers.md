# Server-Side Apply with Unique Field Managers

This proposal is a follow up on the [original Server-Side Apply proposal](./server-side-apply.md), and seeks to make Argo more flexible and give users the ability to apply changes to shared Kuberentes resources at a granular level. It also considers how to remove field-level changes when an Application is destroyed.

To quote the [Kubernetes docs][1]:

> Server-Side Apply helps users and controllers manage their resources through declarative configuration. Clients can create and modify objects declaratively by submitting their _fully specified intent_.

> A fully specified intent is a partial object that only includes the fields and values for which the user has an opinion. That intent either creates a new object (using default values for unspecified fields), or is combined, by the API server, with the existing object.

---

## Open Questions

### [Q-1] What should the behavior be for a server-side applied resource upon Application deletion?

The current behavior is to delete all objects "owned" by the Application. A user could choose to leave the resource behind with `Prune=false`, but that would also leave behind any config added to the shared object. Should the default delete behavior for server-side applied resources be to remove any fields that match that Application's [field manager][2]?

### [Q-2] What sync status should the UI display on a shared resource?

If an Application has successfully applied its partial spec to a shared resource, should it display as "in sync"? Or should it show "out of sync" when there are other changes to the shared object?

## Summary

ArgoCD supports server-side apply, but it uses the same field manager, `argocd-controller`, no matter what Application is issuing the sync. Letting users set a field manager, or defaulting to a unique field manager per application would enable users to:

- Manage only specific fields they care about on a shared resource
- Avoid deleting or overwriting fields that are managed by other Applications

## Motivation

There exist situations where disparate Applications need to add or remove configuration from a shared Kubernetes resource. Server-side apply supports this behavior when different field managers are used.

## Goals

All following goals should be achieve in order to conclude this proposal:

#### [G-1] Applications can apply changes to a shared resource without disturbing existing fields

A common use case of server-side apply is the ability to manage only particular fields on a share Kubernetes resource, while leaving everything else in tact. This requires a unique field manager for each identity that shares the resource.

#### [G-2] Applications that are destroyed only remove the fields they manage from shared resources

A delete operation should undo only the additions or updates it made to a shared resource, unless that resource has `Prune=true`.

#### [G-3] Users can define a field manager as a sync option

Some users may rely on the current behavior emanating from the use of the same field manager across all Applications. The current behavior being that each server side apply sync overwrites all fields on a resource. In other words, "latest sync wins."

## Non-Goals

N/A

## Proposal

Change the Server Side Apply field manager from `argocd-controller` to the Application name. Optionally, allow users to set the field manager to whatever they choose. This will allow Applications to apply partial specs that don't overwrite fields they don't own.

### Use cases

The following use cases should be implemented:

#### [UC-1]: As a user, I would like to manage specific fields on a Kubernetes object shared by other ArgoCD Applications.

Change the Server-Side Apply field manager to be unique to each Application. Eliminate the constant `ArgoCDSSAManager` and replace references with the Application name.

#### [UC-2]: As a user, I would like explict control of which field manager my Application uses for server-side apply.

Add a sync option named `FieldManager` that can be set on the Application or via annotation that controls which field manager is used.

### Security Considerations

TBD

### Risks and Mitigations

#### [R-1] Field manager names are limited to 128 characters

We should trim every field manager to 128 characters.

### Upgrade / Downgrade

TBD

## Drawbacks

- Increase in ArgoCD code base complexity.
- All current sync options are booleans. Adding a `FieldManager` option would be a string.


[1]: https://kubernetes.io/docs/reference/using-api/server-side-apply/
[2]: https://kubernetes.io/docs/reference/using-api/server-side-apply/#managers
