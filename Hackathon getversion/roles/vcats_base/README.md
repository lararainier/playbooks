Role Name
=========

Role Name
=========

This is a role can be used as a basis for VCATS playbooks. It will set up the output files and add the playbook output in a format that's supported by VCATS.

Requirements
------------

* VCATS
* Ansible
* Jinja2

Role Variables
--------------

### defaults/main.yml
Default paths for output files. These output files will be sent back to VCATS and remote systems like DEA, ETMS, etc

* outfile: Main output file. The contents of the file will be parsed by VCATS and remote systems.
* debugfile: Detailed information can be stored in the debugfile.
* errorfile: Any errors should go into the error file.

Other variables:
* vcats_status: The result status of your playbook. Can be set to Success or Failed. Defaults to Success
* vcats_output: The output that will be added to the output file at the end of the playbook. Store your playbook's output in this variable to add it to the output file.
* vcats_output_type: The output type can be direct, vcats or etms. Type direct doesn't apply any formatting, while vcats and etms will add a header and vcats uses a specific format for the data. Defaults to direct.
* ETMS_NEW_CMT: Comment separator for output sent to ETMS. You can use this in your playbook output when you want to add multiple comments to the ETMS ticket.


### vars/main.yml
Here we define variables that have high precedence and aren't intended to be changed by the play.

* outfile: Main output file. Will be parsed by VCATS.
* debugfile: Detailed information can be stored in the debugfile.
* errorfile: Any errors should go into the error file.
* DNSshortname: The DNS shortname (first part of the DNS Entityname), taken from the DNS Entity name that's included in the inventory.
* DNSlocation: The DNS location (second part of the DNS Entityname), taken from the DNS Entity name that's included in the inventory.

VCATS provides a few environmental variables that can be used by a playbook. This role makes these available as regular variables for your playbook.
* VCATS_INV_WS: Path to VCATS working directory. Output files should go here (using subdirectory output/) and any files that were uploaded as part of the request can also be found here.
* VCATS_DEVICE_ATTRS_BASE_URL: The base URL to be used for VCATS API calls


Dependencies
------------

None

Example Playbook
----------------

- hosts: all
  gather_facts: false
  roles:
  # Include this role to setup VCATS output files and create VCATS variables
  - role: vcats_base
    vars:
    # Define the format of the output file. Use vcats, etms or direct. Default setting is direct (no additional formatting will be applied)
    - vcats_output_type: 'direct'

  tasks:
  - block:
    - name: Run 'show version' command on CPE running Cisco IOS
      ios_command:
        commands: show version
      # Register the output of the command under a variable named output
      register: output

    - name: Record command output
      set_fact:
        # If vcats_output_type is set to vcats, set the status to Success or Failed. The status will be published by VCATS.
        # For other output formats, this field is not required
        vcats_status: "Success"
        # Record the output that should be added to the output.dat file for VCATS
        # When the format is set to etms, you can use the variable ETMS_NEW_CMT to insert a line that will split the output into multiple comments
        vcats_output: "Output from {{ ansible_host }}\n---------------------------------------------\n{{ output.stdout.0 }}\n\n"

    rescue:
    # This task will be executed if one of the tasks in the block above fails
    - name: Catch error
      set_fact:
        vcats_status: "Failed"
        vcats_output: "Playbook failed for {{ ansible_host }}\n---------------------------------------------\nSee errorlog for details\n\n"
        # Store the error in the vcats_error variable. The content of this variable will be written to the VCATS errorfile.
        vcats_error: "Failed task ''{{ansible_failed_task.name}}'', {{ansible_failed_result | to_json(ensure_ascii=False) }}"


Author Information
------------------

maarten.oudega@intl.verizon.com

