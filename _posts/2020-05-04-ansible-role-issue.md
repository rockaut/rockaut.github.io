---
layout: post
title: "Issue with Collections/Roles and Ansible 2.9"
subtitle: "*sing* You know it's bad ..."
date: 2020-05-04T18:10:44+02:00
categories: ["favorite-tools", "tools"]
tags: ["Tools", "Favorite Tools", "Ansible", "Open Source"]
readtime: true
---

Over the weekend I stumbled upon a pretty bad issue in Ansible 2.9 with roles in collections.
I contacted Jeff Geerling ([@geerlingguy](https://twitter.com/geerlingguy)) as the allround oracle and, as I forgot about it, he today opened an Issue for it: __[see Github Issue](https://github.com/ansible/ansible/issues/69307)__.

Basically with Ansible 2.9 if you have collections with same named roles (for example 'install') which you include you have to work around the issue or Ansible fails in different ways.

So whats going on in short ...
<!--more-->

---

If you include your roles directly in the play definition, only one of the roles will be run. Even if you use FQCN!
For example:

```yaml
---
- hosts: localhost
  connection: local
  gather_facts: no
  collections:
    - rockaut.collectionA
    - rockaut.collectionB
  roles:
    - rockaut.collectionA.install
    - rockaut.collectionB.install 
```

In this example only `rockaut.collectionA.install` will be executed.

```console
PLAY [localhost] *******************************************************************************************************

TASK [include_role : rockaut.collectionA] ******************************************************************************

TASK [install : debug] *************************************************************************************************
ok: [127.0.0.1] => {
    "msg": "Hello from rockaut.collectionA.install"
}

PLAY RECAP *************************************************************************************************************
127.0.0.1                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Now if you include your roles with `include_role` it fails in a different way. Ansible will run the first mentioned role a second time.
Let's have a look:

```yaml
---
- hosts: localhost
  connection: local
  gather_facts: no
  collections:
    - rockaut.collectionA
    - rockaut.collectionB
  tasks:
    # Beware that I switched the order of include just to make a point!
    - include_role:
        name: rockaut.collectionB.install
    - include_role:
        name: rockaut.collectionA.install 
```

Here only `rockaut.collectionB.install` will be executed but two times!

```console
PLAY [localhost] *******************************************************************************************************

TASK [include_role : rockaut.collectionB] ******************************************************************************

TASK [install : debug] *************************************************************************************************
ok: [127.0.0.1] => {
    "msg": "Hello from rockaut.collectionB.install"
}

TASK [include_role : rockaut.collectionB] ******************************************************************************

TASK [install : debug] *************************************************************************************************
ok: [127.0.0.1] => {
    "msg": "Hello from rockaut.collectionB.install"
}

PLAY RECAP *************************************************************************************************************
127.0.0.1                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

So how to work around quickly? Well, it seams that it's fixed in devel branch (I haven't tried it yet) - so with 2.10 it should be gone.
Until then you best way might be to split the execution in separate plays.

```yaml
---
- hosts: localhost
  connection: local
  gather_facts: no
  collections:
    - rockaut.collectionA
  tasks:
    - include_role:
        name: rockaut.collectionA.install

- hosts: localhost
  connection: local
  gather_facts: no
  collections:
    - rockaut.collectionB
  roles:
    - rockaut.collectionB.install
```

Which is now how we wanted it in the first place:

```console
PLAY [localhost] *******************************************************************************************************

TASK [include_role : rockaut.collectionA] ******************************************************************************

TASK [install : debug] *************************************************************************************************
ok: [127.0.0.1] => {
    "msg": "Hello from rockaut.collectionA.install"
}

TASK [include_role : rockaut.collectionB] ******************************************************************************

TASK [install : debug] *************************************************************************************************
ok: [127.0.0.1] => {
    "msg": "Hello from rockaut.collectionB.install"
}

PLAY RECAP *************************************************************************************************************
127.0.0.1                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
