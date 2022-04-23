# ansible-issue-69677
This a test for [Ansible issue #69677](https://github.com/ansible/ansible/issues/69677) (Per var merging of lists (and hashes?)).

The expectation is for variables **mergeable_list** and **mergeable_dict** (see [the *'all'* group](./inventories/common-all)) to show the result of summing up the contents of *all* and *somegroup* values, since *localhost* belongs to both groups.

It instead dumps an unhandled exception because of *'recursive loop detected in template string'*.

## usage
This repository's development environment is managed by means of my [ansible-issues](https://github.com/jmnavarrol/ansible-issues) project, so for easy use I recommend cloning/forking it.

Otherwise is up to you to configure your running environment as you please.

**Actions:**
1. **What works:** You, of course, can *merge/combine* inventory variables (either *lists* or *dictionaries*) provided you know in advance the involved names.  
  Run [the test playbook](./playbook.yml) along [the working inventory](./inventories/doeswork/):
    ```console
    (ansible-issue-69677) user@host:~/ansible-issue-69677$ ansible-playbook -i inventories/doeswork/ playbook.yml
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.7.3 (default, Jan 22 2021, 20:04:44) 
    [GCC 8.3.0]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
    
    PLAY [A test for #69677. See https://github.com/ansible/ansible/issues/69677] ******************************************************************************************
    
    TASK [Gathering Facts] *************************************************************************************************************************************************
    ok: [localhost]
    
    TASK [Shows involved versions] *****************************************************************************************************************************************
    ok: [localhost] => {
        "msg": [
            "Python version: '3.7.3'.",
            "Ansible version '2.11.3'."
        ]
    }
    
    TASK [Original variables] **********************************************************************************************************************************************
    ok: [localhost] => {
        "msg": [
            "DOES WORK:",
            "original list: ['original-value']",
            "original dict: {'original-key': 'original-value'}"
        ]
    }
    
    TASK [Merges inventory variables with different names.] ****************************************************************************************************************
    ok: [localhost] => {
        "msg": [
            "DOES WORK:",
            "merged lists: ['original-value', 'another-value']",
            "merged dicts: {'original-key': 'original-value', 'another-key': 'another-value'}"
        ]
    }
    
    TASK [Tries to overload inventory variables.] **************************************************************************************************************************
    skipping: [localhost]
    
    PLAY RECAP *************************************************************************************************************************************************************
    localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
    
    (ansible-issue-69677) user@host:~/ansible-issue-69677$
    ```

1. **What does NOT work:** Trying to *"overload"* an already existing list/dict (i.e. so different groups can work independently and still share variable results) fails with *recursive loop detected in template string* with. Run [the test playbook](./playbook.yml) along [the FAILING inventory](./inventories/doesnotwork/) and see the error:
    ```console
    (ansible-issue-69677) user@host:~/ansible-issue-69677$ ansible-playbook -i inventories/doesnotwork/ playbook.yml
    [DEPRECATION WARNING]: Ansible will require Python 3.8 or newer on the controller starting with Ansible 2.12. Current version: 3.7.3 (default, Jan 22 2021, 20:04:44) 
    [GCC 8.3.0]. This feature will be removed from ansible-core in version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
    
    PLAY [A test for #69677. See https://github.com/ansible/ansible/issues/69677] ******************************************************************************************
    
    TASK [Gathering Facts] *************************************************************************************************************************************************
    ok: [localhost]
    
    TASK [Shows involved versions] *****************************************************************************************************************************************
    ok: [localhost] => {
        "msg": [
            "Python version: '3.7.3'.",
            "Ansible version '2.11.3'."
        ]
    }
    
    TASK [Original variables] **********************************************************************************************************************************************
    skipping: [localhost]
    
    TASK [Merges inventory variables with different names.] ****************************************************************************************************************
    skipping: [localhost]
    
    TASK [Tries to overload inventory variables.] **************************************************************************************************************************
    fatal: [localhost]: FAILED! => {"msg": "An unhandled exception occurred while templating '{{ mergeable_list | union(['some_other']) }}'. Error was a <class 'ansible.err
    (...)
    geable_list | union(['some_other']) }}'. Error was a <class 'ansible.errors.AnsibleError'>, original message: recursive loop detected in template string: {{ mergeable_list | union(['some_other']) }}"}    
    PLAY RECAP *************************************************************************************************************************************************************
    localhost                  : ok=2    changed=0    unreachable=0    failed=1    skipped=2    rescued=0    ignored=0   
    
    (ansible-issue-69677) user@host:~/ansible-issue-69677$
    ```
