---
title: Backup and Restore of Kong Gateway's Configuration
content_type: how-to
---

You can use decK to back up and restore a subset or the entirety of
{{site.base_gateway}}'s entity configuration. 

See the reference for [Entities Managed by decK](/deck/{{page.kong_version}}/reference/entities/)
to find out which entity configurations can be backed up.

## Back up {{site.base_gateway}}'s entire configuration

To back up {{site.base_gateway}}'s configuration, use the `dump` command:

```sh
deck dump
```
This generates a `kong.yaml` file with the entire configuration of {{site.base_gateway}}, if possible. 

Then, restore this file back to {{site.base_gateway}} using the `sync` command:

1. Preview the changes that decK will perform:
    
    ```sh
    deck diff
    ```

2. Re-create the entities in {{site.base_gateway}}:
    
    ```sh
    deck sync
    ```

## Manage a subset of configuration

You can export, import, and manage a subset of {{site.base_gateway}}'s configuration using decK's
`select-tag` feature. This is similar to adopting
[distributed configuration](/deck/{{page.kong_version}}/guides/distributed-configuration) for {{site.base_gateway}}.

The `select-tag` feature assumes that all the entities you would like to manage
in {{site.base_gateway}} share a common tag(s).

Assuming you have such a common tag (for example, let's call it `foo-tag`),
you can use it to export only a subset of the configuration:

```sh
deck dump --select-tag foo-tag
```

If you observe the file generated by decK, you will see the following section:

```yaml
_info:
  select_tags:
  - foo-tag
```

This subsection tells decK to filter out entities containing select-tags during
a sync operation.

Now, you can manage or sync back only this subset of {{site.base_gateway}}'s configuration.
