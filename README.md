# 389ds-replication

[![Ansible Galaxy](https://img.shields.io/ansible/role/40566.svg)](https://galaxy.ansible.com/lvps/389ds_replication)

Configure replication between 389DS server (LDAP server) instances.

```shell
ansible-galaxy install lvps.389ds_replication
```

## Requirements

- Ansible version: 2.7 or higher
- OS: CentOS 7

## Role Variables

| Variable                                | Default                                                                        | Description                                                                                                                                                                                                                                                                                                                                                                                                              | Can be changed | Role |
|-----------------------------------------|--------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------|------|
| dirsv_role                              |                                                                                | Role of this server: 'supplier', 'consumer' or 'both' (master). Hub is not supported.                                                                                                                                                                                                                                                                                                                                    | **No**         |      |
| dirsrv_uri                              | "ldap://localhost"                                                             | URI of the server to configure. Since this runs on the Ansible target, localhost should be fine. It's possible to set it to `ldaps://localhost` to use TLS on port 636.                                                                                                                                                                                                                                                  |                | CSB  |
| dirsrv_rootdn                           | "cn=Directory Manager"                                                         | Root DN ("administrator" account username)                                                                                                                                                                                                                                                                                                                                                                               |                | CSB  |
| dirsrv_rootdn_password                  | secret                                                                         | Password for root DN account                                                                                                                                                                                                                                                                                                                                                                                             |                | CSB  |
| dirsrv_use_starttls                     | true                                                                           | Use StartTLS to connect to the server                                                                                                                                                                                                                                                                                                                                                                                    |                | CSB  |
| dirsrv_tls_certificate_trusted          | true                                                                           | True if TLS certificate is from a trusted CA, false if self-signed or from a private CA, unused if TLS is not used                                                                                                                                                                                                                                                                                                       |                | CSB  |
| dirsrv_serverid                         | default                                                                        | Server ID aka instance ID, e.g. if the server is installed in the dirsrv/slapd-example directory, "example" is the server ID                                                                                                                                                                                                                                                                                             |                | CSB  |
| dirsrv_suffix                           | dc=example,dc=local                                                            | Root suffix                                                                                                                                                                                                                                                                                                                                                                                                              |                | CSB  |
| dirsrv_supplier_replica_id              | 1                                                                              | Choose a number between 1 and 65534. Don't assign it to other servers or bad things may happen.                                                                                                                                                                                                                                                                                                                           | **No**         | SB   |
| dirsrv_consumer_uri                     | "ldap://consumer.example.com:389/"                                             | Full URI, including port, that the supplier will connect to and perform replication by pushing changes                                                                                                                                                                                                                                                                                                                   | **No**         | SB   |
| dirsrv_replication_user_remote          | Replication Manager                                                            | User account that exists on the consumer. The supplier will bind with this account to perform replication. "Replication Manager" means that the account is "cn=Replication Manager,cn=config"                                                                                                                                                                                                                            | Yes            | SB   |
| dirsrv_replication_user_password_remote |                                                                                | Password for the replication user (Replication Manager) account                                                                                                                                                                                                                                                                                                                                                          | Yes            | SB   |
| dirsrv_replica_bind_method              | "PLAIN"                                                                        | Bind method that supplier uses to connect to the consumer (SIMPLE, PLAIN, SASL)                                                                                                                                                                                                                                                                                                                                          | Yes            | SB   |
| dirsrv_changelog_max_age                | "10d"                                                                          | Sets the value of `nsslapd-changelogmaxage`                                                                                                                                                                                                                                                                                                                                                                              | Yes            | SB   |
| dirsrv_replica_attributes_list          | "(objectclass=*) $ EXCLUDE authorityRevocationList accountUnlockTime memberof" | Sets the value of `nsds5ReplicatedAttributeList`, the default of this variable is use throughout examples in the documentation                                                                                                                                                                                                                                                                                           | Yes            | SB   |
| dirsrv_replica_attributes_list_total    | "(objectclass=*) $ EXCLUDE accountUnlockTime"                                  | Sets the value of `nsds5ReplicatedAttributeListTotal`, the default of this variable is use throughout examples in the documentation                                                                                                                                                                                                                                                                                      | Yes            | SB   |
| dirsrv_replication_user                 | Replication Manager                                                            | User account to create on the consumer. This account will be used by the supplier to bind on this server (the consumer). "Replication Manager" means that the account will be created at "cn=Replication Manager,cn=config"                                                                                                                                                                                              | Yes            | CB   |
| dirsrv_replication_user_password        |                                                                                | Password for that account.                                                                                                                                                                                                                                                                                                                                                                                               | Yes            | CB   |
| dirsrv_begin_replication_immediately    | true                                                                           | Boolean, sets `nsds5ReplicaEnabled` to "on" or "off" in the replication agreement. This should be a safe: if you add a new server it won't start pushing its empty database to other servers in any case because they have a different generation ID and replication fails (see examples for more details), but if you want to be even safer or make some customizations to the replication agreement, set this to false | **No**         | CB   |
| dirsrv_consumer_referral_to_supplier    | "ldap://supplier.example.com:389/"                                             | Full LDAP URI including port. When a client tries to write to a consumer, which is read-only, it will redirect the client to this server (a supplier that can accept writes).                                                                                                                                                                                                                                            | Yes            | C    |

First of all choose if the server is a supplier, consumer or both and set
dirsv_role accordingly. Then set the variables related to that: look in the Role
column, C = Consumer, S = Supplier, B = Both.

Some variables cannot be changed once they are set, changing them will produce
unexcpected results ranging from "nothing happens" to "the role fails". Some
others (authentication details, suffix, etc...) should be set to the correct
value for the server, can be changed as long as that makes sense, i.e. you can
change `dirsrv_rootdn_password` if you've changed the root DN password so that
this role can authenticate correctly, but changing `dirsrv_suffix` between one
run and the other on the same server is pointless, unless you somehow managed to
change the suffix in 389DS.

The following variables have the exact same name and meaning as in the
[389ds-server](https://github.com/lvps/389ds-server) role, so if you're using
both roles in the same playbook you can define them just once:

* dirsrv_rootdn
+ dirsrv_rootdn_password
* dirsrv_tls_certificate_trusted
* dirsrv_serverid
* dirsrv_suffix

## Dependencies

None.

But do keep in mind that this role expects 389DS to be already up and running,
it only configures replication between existing servers.

## Example Playbook

More examples, including 389DS installation from the ground up and the needed
Vagrant configs to test them are available in the [389ds-examples](https://github.com/lvps/389ds-examples/)
repository.

Note that, usually, replication doesn't start immediately because the
["replica generation" is different](https://github.com/colbyprior/389-ldap-server/pull/1#issuecomment-442694193)
between the servers. This can be fixed with the "replica refresh" procedure,
which is described for example in [section 15.2.5](https://access.redhat.com/documentation/en-us/red_hat_directory_server/10/html/administration_guide/managing_replication-configuring-replication-cmd#Configuring-Replication-InitializingConsumers-cmd)
of the Administration Guide. You will need to perform it on suppliers or
servers with role "both".

Basically, doing a "replica refresh" will forcefully push the database from one
supplier to a consumer, replacing the previous consumer database: do this
starting from one supplier and gradually moving toward the others, being careful
to not replace a production database with the empty database of a server that
has just been installed. When two servers have the exact same database,
replication starts automatically and immediately.

The procedure is also exaplained in more details in the [389ds-examples](https://github.com/lvps/389ds-examples/#starting-replication)
repository.

### Consumer and supplier

Configure the consumer first, this server will contain a read-only copy of the
database:

```yaml
- hosts: consumer
  become: true

  roles:
    -
      role: lvps.389ds_replication
      dirsrv_replica_role: consumer
      dirsrv_suffix: "dc=example,dc=local"
      dirsrv_uri: "ldap://localhost"
      dirsrv_rootdn_password: secret
      dirsrv_replication_user_password: foo # Will create cn=Replication Manager,cn=config with this password
      dirsrv_consumer_referral_to_supplier: "ldap://supplier.example.local:389/"
```

The configure the supplier, this server will accept writes and push all changes
to the consumer:

```yaml
- hosts: supplier
  become: true

  roles:
    -
      role: lvps.389ds_replication
      dirsrv_replica_role: supplier
      dirsrv_suffix: "dc=example,dc=local"
      dirsrv_uri: "ldap://localhost"
      dirsrv_rootdn_password: verysecret
      dirsrv_replication_user_password_remote: foo # Will bind with cn=Replication Manager,cn=config and this password on the other server
      dirsrv_consumer_uri: "ldap://consumer.example.local:389/" # The other server (the consumer defined above)
      dirsrv_supplier_replica_id: 123
```

### Multi-master with two masters

```yaml
- hosts: mm1
  become: true
  roles:
    -
      role: lvps.389ds_replication
      dirsrv_replica_role: 'both'
      dirsrv_suffix: "dc=example,dc=local"
      dirsrv_uri: "ldap://localhost"
      dirsrv_rootdn_password: secret1
      dirsrv_replication_user_password: "aaaaaa"
      dirsrv_replication_user_password_remote: "bbbbbb" # On the other server
      dirsrv_consumer_uri: "ldap://mm2.example.local:389/" # The other server
      dirsrv_supplier_replica_id: 1
```

```yaml
- hosts: mm2
  become: true
  roles:
    -
      role: lvps.389ds_replication
      dirsrv_replica_role: 'both'
      dirsrv_suffix: "dc=example,dc=local"
      dirsrv_uri: "ldap://localhost"
      dirsrv_rootdn_password: secret2
      dirsrv_replication_user_password: "bbbbbb"
      dirsrv_replication_user_password_remote: "aaaaaa" # On the other server
      dirsrv_consumer_uri: "ldap://mm1.example.local:389/" # The other server
      dirsrv_supplier_replica_id: 2
```

## Known bugs

If `dirsrv_replication_user_password` is changed, no change is reported: this
is because password actually changes on every run (Ansible can't tell if the
previous hashed password is the same as the new one, so it will be
changed and hashed again), but there's a `changed_when: false` to hide that
detail.

## License

MIT.
