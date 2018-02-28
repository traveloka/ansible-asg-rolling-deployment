# blue_green-lc-deploy

Update an AWS auto scaling group

## Requirements
 - [ec2_ami_find](http://docs.ansible.com/ansible/latest/ec2_ami_find_module.html#requirements-on-host-that-executes-module)
 - [ec2_lc](http://docs.ansible.com/ansible/latest/ec2_lc_module.html#requirements-on-host-that-executes-module)
 - [ec2_asg](http://docs.ansible.com/ansible/latest/ec2_asg_module.html#requirements-on-host-that-executes-module)

## Role Variables
 - aws_role_arn: the ARN of an AWS IAM role this role will assume
 - new_ami_owner: the ami owner filter to apply to search for the new ami
 - new_ami_name_filter: the ami name filter to apply to search for the new ami
 - new_ami_id: the ami id to create a launch configuration with
 - other either straightforward or traveloka-specific variables

## Dependencies

 - The AWS EC2 ASG is already provisioned

## Convention

 - The AWS EC2 AMI must have these tags:
   - Service
   - ServiceVersion
   - ProductDomain
   - Application
   - BaseAmiId

## Example Playbook

```
---
- hosts: localhost
  connection: local
  become: yes
  become_user: root
  become_method: sudo
  become_flags: -E
  vars:
    # set common_java_opts in supervisor using the ami baking playbook!
    app_memory_java_opts: >-
      -Xms1g -Xmx1g -XX:PermSize=512m -XX:MaxPermSize=512m
  roles:
    - role: "ansible-blue_green-lc-deploy"
      infra_environment: staging
      instance_type: t2.small
      instance_profile: traveloka-flight-instance-profile
      instance_sgs:
        - sg-a2b80f3c
      instance_user_data: |
        #cloud-config
        bootcmd:
        - echo "Traveloka dulu"
      asg_name: traveloka-flight-asg-01
      asg_wait_timeout: 30
      aws_region: ap-southeast-1
      aws_role_arn: arn:aws:iam::012345678901:role/traveloka-flight-deploy
      new_ami_name_filter: traveloka-flight-*
```

## License

MIT

## Author Information

[Salvian Reynaldi](https://github.com/SalzzZ)
