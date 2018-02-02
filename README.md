# blue_green-lc-deploy

Update an AWS auto scaling group

## Requirements
 -  [ec2_ami_find](http://docs.ansible.com/ansible/latest/ec2_ami_find_module.html#requirements-on-host-that-executes-module)
 - [ec2_lc](http://docs.ansible.com/ansible/latest/ec2_lc_module.html#requirements-on-host-that-executes-module)
 - [ec2_asg](http://docs.ansible.com/ansible/latest/ec2_asg_module.html#requirements-on-host-that-executes-module)

## Role Variables
 - aws_role_arn: the ARN of an AWS IAM role this role will assume
 - aws_role_session_name: the name that will be used during the assume-role session, default to "ansible"
 - new_ami_owner: the ami owner filter to apply to search for the new ami
 - new_ami_name_filter: the ami name filter to apply to search for the new ami
 - new_ami_id: the ami id to create a launch configuration with
 - other either straightforward or traveloka-specific variables

## Dependencies

None

## Example Playbook

```
- hosts: localhost
  connection: local
  vars:
    aws_region: ap-southeast-1
    aws_role_arn: arn:aws:iam::123456789012:role/AnsibleDeployRole-{{ app_name }}-{{ infra_environment }}
    new_ami_name_filter: traveloka/{{ app_name }}-*
    app_name: traveloka_flight_booking
    app_product_domain: flight
    app_environment: staging
    config_env: custom_staging
    app_start_java_opts: -Xms1g -Xmx1g -XX:PermSize=512m -XX:MaxPermSize=512m
    infra_environment: staging
    asg_name: "traveloka_flight_booking_asg
    new_ami_id: "ami-a1b2c3d4"
    asg_health_check_type: ELB
    asg_health_check_period: 40
    asg_target_group_arns:
      - arn:aws:elasticloadbalancing:ap-southeast-1:123456789012:targetgroup/traveloka_flight_booking_main_target_group/fj192h5l12u29fnj
    asg_wait_timeout: 120
    instance_type: t2.medium
    instance_sgs:
      - sg-b4a8c92d
    asg_vpc_zone_identifier:
      - subnet-91ea5c5b
      - subnet-7a6b3e4f
  roles:
    - role: "ansible-blue_green-lc-deploy"

```

## License

MIT

## Author Information

[Salvian Reynaldi](https://github.com/SalzzZ)
