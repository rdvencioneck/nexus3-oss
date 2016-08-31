# Ansible Role: Nexus 3 OSS

This role installs and configures Nexus Repository Manager OSS version 3.x on CentOS/RHEL.

## Requirements

- This has only been tested on CentOS 7
- Oracle Java 8 (mandatory)
- Apache HTTPD (optional, used to setup a SSL reverse-proxy)

(see [Dependencies](#dependencies) section below for matching roles on galaxy)

## Role Variables


Ansible variables, along with the default values (see `default/main.yml`) :

    nexus_version: '3.0.1-01'
    nexus_package: "nexus-{{ nexus_version }}-unix.tar.gz"

The nexus version and package to install, see available versions at https://www.sonatype.com/download-oss-sonatype .

    nexus_download_dir: '/tmp'

Directory on target where the nexus package will be downloaded.

    nexus_os_group: 'nexus'
    nexus_os_user: 'nexus'

User and group used to own the nexus files and run the service, those will be created by the role if absent.

    nexus_installation_dir: '/opt'
    nexus_data_dir: '/var/nexus'
    nexus_tmp_dir: '/tmp/nexus'

Nexus directories, `nexus_installation_dir` contains the installed executable(s), `nexus_data_dir` contains all configuration, repositories and uploaded artifacts.

    nexus_admin_password: 'changeme'

The 'admin' account password to setup. Note : admin password change subsequent to first-time provisionning/install is *not implemented* by this role yet.

    nexus_anonymous_access: false

Allow [anonymous access](https://books.sonatype.com/nexus-book/3.0/reference/security.html#anonymous) to nexus.

    public_hostname: 'nexus.vm'

The fully qualified domain name under which the nexus instance will be accessible to its clients.

    httpd_setup_enable: false
    httpd_ssl_certificate_file: 'files/nexus.vm.crt'
    httpd_ssl_certificate_key_file: 'files/nexus.vm.key'

Setup an [SSL Reverse-proxy](https://books.sonatype.com/nexus-book/3.0/reference/install.html#_example_reverse_proxy_ssl_termination_at_base_path), this needs httpd installed.

    ldap_connections: []

[LDAP connection(s)](https://books.sonatype.com/nexus-book/3.0/reference/security.html#ldap) setup, each item goes as follow :

```
  - ldap_name: 'My Company LDAP' # used as a key to update the ldap config
    ldap_protocol: 'ldaps' # ldap or ldaps
    ldap_hostname: 'ldap.mycompany.com'
    ldap_port: 636
    ldap_search_base: 'dc=mycompany,dc=net'
    ldap_user_base_dn: 'ou=users'
    ldap_user_object_class: 'inetOrgPerson'
    ldap_user_id_attribute: 'uid'
    ldap_user_real_name_attribute: 'cn'
    ldap_user_email_attribute: 'mail'
    ldap_group_base_dn: 'ou=groups'
    ldap_group_object_class: 'posixGroup'
    ldap_group_id_attribute: 'cn'
    ldap_group_member_attribute: 'memberUid'
    ldap_group_member_format: '${username}'
```


    nexus_privileges:
      - name: all-repos-read # used as key to update a privilege
        description: 'Read & Browse access to all repos'
        repository: '*'
        actions: # can be add, browse, create, delete, edit, read or  * (all)
          - read
          - browse

List of the [privileges](https://books.sonatype.com/nexus-book/3.0/reference/security.html#privilegeshttps://books.sonatype.com/nexus-book/3.0/reference/security.html#privileges) to setup. Those items are combined with the following default values :

```
    _nexus_privilege_defaults:
      type: repository-view
      format: maven2
      actions:
        - read
```

```
    nexus_roles:
      - id: Developpers # can map to a LDAP group id, also used as a key to update a role
        name: developers
        description: All developers
        privileges:
          - nx-search-read
          - all-repos-read
        roles: [] # references to other role names
```

List of the [roles](https://books.sonatype.com/nexus-book/3.0/reference/security.html#roles) to setup.

    nexus_delete_default_repos: true

Delete the repositories from the nexus install initial default configuration. This step is only executed on first-time install (when `nexus_data_dir` has been detected empty).

    nexus_repos_maven_proxy:
      - name: central
        remote_url: 'https://repo1.maven.org/maven2/'
        layout_policy: permissive
      - name: jboss
        remote_url: 'https://repository.jboss.org/nexus/content/groups/public-jboss/'
    # example with a login/password :
    # - name: secret-remote-repo
    #   remote_url: 'https://company.com/repo/secure/private/go/away'
    #   remote_username: 'username'
    #   remote_password: 'secret'

Maven [proxy repositories](https://books.sonatype.com/nexus-book/3.0/reference/maven.html#_proxying_maven_repositories) configuration.


    nexus_repos_maven_hosted:
      - name: private-release
        version_policy: release
        write_policy: allow_once

Maven [hosted repositories](https://books.sonatype.com/nexus-book/3.0/reference/maven.html#_hosting_maven_repositories) configuration.

    nexus_repos_maven_group:
      - name: public
        member_repos:
          - central
          - jboss

Maven [group repositories](https://books.sonatype.com/nexus-book/3.0/reference/maven.html#_grouping_maven_repositories) configuration.

All three repository types are combined with the following default values :

```
    _nexus_repos_maven_defaults:
      blob_store: default
      strict_content_validation: true
      version_policy: release # release, snapshot or mixed
      layout_policy: strict # strict or permissive
      write_policy: allow_once # allow_once or allow
```


## Dependencies

This role requires Ansible 2.1 or higher.

The java and httpd requirements /can/ be fulfilled with the following galaxy roles :
  - [ansiblebit.oracle-java](https://galaxy.ansible.com/ansiblebit/oracle-java/)
  - [geerlingguy.apache](https://galaxy.ansible.com/geerlingguy/apache/)

## Example Playbook

```
---
- name: Nexus
  hosts: nexus
  become: yes

  vars:
    nexus_version: '3.0.1-01'
    nexus_admin_password: "{{ vault_nexus_admin_password }}"
    httpd_server_name: 'nexus.vm'
    httpd_setup_enable: true
    httpd_ssl_certificate_file: "{{ vault_httpd_ssl_certificate_file }}"
    httpd_ssl_certificate_key_file: "{{ vault_httpd_ssl_certificate_key_file }}"
    ldap_connections:
      - ldap_name: 'Company LDAP'
        ldap_protocol: 'ldaps'
        ldap_hostname: 'ldap.company.com'
        ldap_port: 636
        ldap_search_base: 'dc=company,dc=net'
        ldap_user_base_dn: 'ou=users'
        ldap_user_object_class: 'inetOrgPerson'
        ldap_user_id_attribute: 'uid'
        ldap_user_real_name_attribute: 'cn'
        ldap_user_email_attribute: 'mail'
        ldap_group_base_dn: 'ou=groups'
        ldap_group_object_class: 'posixGroup'
        ldap_group_id_attribute: 'cn'
        ldap_group_member_attribute: 'memberUid'
        ldap_group_member_format: '${username}'
    nexus_privileges:
      - name: all-repos-read
        description: 'Read & Browse access to all repos'
        repository: '*'
        actions:
          - read
          - browse
      - name: company-project-deploy
        description: 'Deployments to company-project'
        repository: company-project
        actions:
          - add
          - edit
    nexus_roles:
      - id: Developpers # maps to the LDAP group
        name: developers
        description: All developers
        privileges:
          - nx-search-read
          - all-repos-read
          - company-project-deploy
        roles: []
    nexus_repos_maven_proxy:
      - name: central
        remote_url: 'https://repo1.maven.org/maven2/'
        layout_policy: permissive
      - name: alfresco
        remote_url: 'https://artifacts.alfresco.com/nexus/content/groups/private/'
        remote_username: 'secret-username'
        remote_password: "{{ vault_alfresco_private_password }}"
      - name: jboss
        remote_url: 'https://repository.jboss.org/nexus/content/groups/public-jboss/'
      - name: vaadin-addons
        remote_url: 'https://maven.vaadin.com/vaadin-addons/'
      - name: jaspersoft
        remote_url: 'https://jaspersoft.artifactoryonline.com/jaspersoft/jaspersoft-repo/'
        version_policy: mixed
    nexus_repos_maven_hosted:
      - name: company-project
        version_policy: mixed
        write_policy: allow
    nexus_repos_maven_group:
      - name: public
        member_repos:
          - central
          - jboss
          - vaadin-addons
          - jaspersoft

  roles:
    - role: ansiblebit.oracle-java
      oracle_java_set_as_default: yes
    - role: geerlingguy.apache
      apache_create_vhosts: no
    - role: ansible-nexus

```

License
-------

GNU GPLv3

Author Information
------------------

See https://github.com/savoirfairelinux/ansible-nexus3-oss