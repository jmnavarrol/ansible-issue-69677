# ansible-issue-69677
This a test for [Ansible issue #69677](https://github.com/ansible/ansible/issues/69677).

It's been tested using the Python *virtualenv* defined on [ansible.requirements](./ansible.requirements) and Python 2.7.13 (but I don't think using Python 3 instead makes any difference).

The expectation is for variables **merged_list** and **merged_dict** as defined for [somegroup](./ansible/inventory/group_vars/somegroup) show the result of summing up the contents of both the *all* and *somegroup* values, since *localhost* belongs to both groups.

It instead dumps an unhandled exception because of *'recursive loop detected in template string'*.

## usage
1. Create and load a Python virtualenv with [ansible.requirements](./ansible.requirements) (I used [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io)).
1. Run ansible on this repo's root against (the test playbook)[./ansible/playbook.yml] (result *as-is*):
   ```console
   (ansible) user@host:~/ansible-issue-69677$ ansible-playbook -i ansible/inventory/ ansible/playbook.yml 
   
   PLAY [A test for #69677] ***************************************************************************************************************************************************************************
   
   TASK [Gathering Facts] *****************************************************************************************************************************************************************************
   [WARNING]: Platform linux on host localhost is using the discovered Python interpreter at /usr/bin/python, but future installation of another Python interpreter could change this. See
   https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.
   ok: [localhost]
   
   TASK [A test for #69677] ***************************************************************************************************************************************************************************
   ok: [localhost] => {
       "msg": [
           "merged_list: []", 
           "different_merged_list: ['some_other']", 
           "merged_dict: {}", 
           "different_merged_dict: {'foo': 'foo_value'}"
       ]
   }
   
   PLAY RECAP *****************************************************************************************************************************************************************************************
   localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
   
   (ansible) user@host:~/ansible-issue-69677$
   ```
1. Now, uncomment vars at [somegroup](./ansible/inventory/group_vars/somegroup), re-run the command above and see the error.
