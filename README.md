## BIGIP AS3 ansible templating tool

Used for declaratively managing BIGIP configuration using AS3.
It is not meant to be a fully complete AS3 templating tool. This is meant more of a base for your various application templates and to get you started in automating BIGIPs.

### Templates

The `bigip_as3_gen` role uses application templates (three are already provided (`https`, `dns`, `forward`)) to generate AS3 configuration using ansible variables with Jinja2.

You should be able to create your own templates easily under [roles/bigip_as3_gen/templates/app/templates](roles/bigip_as3_gen/templates/app/templates) using some of the already defined AS3 objects (some of the configuration may be different from what is specified in AS3 schema reference but it is kept as close as possible).

### Requirements

- `ansible-core==2.11.1`
- `jsonschema==3.2.0`
- `collections:`
  - `name: ansible.utils`
    `version: 2.2.0`

#### Usage:

- Run `ansible-playbook gen_playbook.yml` to generate AS3 configs in `backup/` directory (directory is auto generated) in JSON and YAML format
- Exchange correct credentials for BIGIP in `inventory.yml` and run `ansible-playbook deploy_playbook.yml` to deploy generated configuration to BIGIPs defined in inventory.yml

#### Updating the validation schema
The validation schema currently uses 3.26.1 version and it can be updated from [AS3 repo](https://github.com/F5Networks/f5-appsvcs-extension/tree/master/schema/latest).
After replacing it in `roles/bigip_as3_gen/as3-schema.json`, you have to remove the `$id`, as it seems that having `URN` in `$id` causes errors, see [this issue](https://github.com/Julian/jsonschema/issues/544)

### Reference
Can be found under [REFERENCE.md](REFERENCE.md)
