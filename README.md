# blue_green-lc-deploy #

Ansible Role that can be used either for doing Rolling Deployment on the ASG or just updating some ASG parameters.

## Requirements ##

- Python >= 2.7 
- boto3 == 1.8.2
- ansible == 2.5.8

It is recommended to use [virtualenv](https://virtualenv.pypa.io/en/stable/installation/) and [pip](https://pip.pypa.io/en/stable/installing/) to install both [boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#installation) and [ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#latest-releases-via-pip).

Requirements mentioned above are based on these modules which are used in this role:
- [debug](https://docs.ansible.com/ansible/2.5/modules/debug_module.html) - Print statements during execution
- [fail](https://docs.ansible.com/ansible/2.5/modules/fail_module.html) - Fail with custom message
- [ec2_ami_facts](https://docs.ansible.com/ansible/2.5/modules/ec2_ami_facts_module.html) - Gather facts about ec2 AMIs 
- [ec2_asg](https://docs.ansible.com/ansible/2.5/modules/ec2_asg_module.html) - Create or delete AWS Autoscaling Groups
- [ec2_asg_facts](https://docs.ansible.com/ansible/2.5/modules/ec2_asg_facts_module.html) - Gather facts about ec2 Auto Scaling Groups (ASGs) in AWS
- [ec2_lc](https://docs.ansible.com/ansible/2.5/modules/ec2_lc_module.html) - Create or delete AWS Autoscaling Launch Configurations
- [ec2_lc_facts](https://docs.ansible.com/ansible/2.5/modules/ec2_lc_facts_module.html) - Gather facts about AWS Autoscaling Launch Configurations
- [set_fact](https://docs.ansible.com/ansible/2.5/modules/set_fact_module.html) - Set host facts from a task
- [sts_assume_role](https://docs.ansible.com/ansible/2.5/modules/sts_assume_role_module.html) - Assume a role using AWS Security Token Service and obtain temporary credentials


## Role Variables ##

### Defaults ###
 
    - name: asg_wait_timeout
      description: How long in seconds to wait for instances to become viable when replaced.
      value: 3600
    - name: ebs_device_name
      description: The name of the volume to mount.
      value: /dev/sda1
    - name: ebs_volume_size
      description: The size of the volume in gigabytes.
      value: 8
    - name: ebs_volume_type
      description: The type of volume.
      value: gp2
    - name: ebs_delete_on_termination
      description: Whether the volume should be destroyed on instance termination.
      value: true
    - name: launch_configuration_name_suffix
      description: Suffix that will be added to Launch Configurations which are going to be created and deleted by this role.
      value: rolling
    - name: canary_release_count
      description: No of instance to be updated automatically with new AMI for canary release (Should be strictly less than ASG size) .
      value: 0
    - name: canary_release_instance_ids
      description: Instance ids to be updated manually with new AMI for canary release (Should be strictly less than ASG size) .
      value: []

### Required Variables ###

    - name: aws_region
      description: The AWS region to use.
    - name: ami_id
      description: The AMI unique identifier to be used for new Launch Configuration.
    - name: asg_name
      description: The name of the Autoscaling Group that you are going to modify.
    - name: service_name
      description: The name of the service.
    - name: cluster_role
      description: The role/function of the cluster.
    - name: service_environment
      description: The environment of this service belongs to.
    - name: instance_user_data
      description: Several commands that you want to execute on the instance when the instance is launched. Read more: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html

### Optional Variables ###

    - name: asg_max_size
      description: Maximum number of instances in group, if unspecified then the current group value will be used.
    - name: asg_min_size
      description: Minimum number of instances in group, if unspecified then the current group value will be used.
    - name: asg_desired_capacity
      description: Desired number of instances in group, if unspecified then the current group value will be used.
    - name: asg_replace_batch_size
      description: Number of instances you'd like to replace at a time, if unspecified then the default value is 1.
    - name: asg_health_check_type
      description: Controls how health checking is done, if unspecified then the current group value will be used.
    - name: asg_health_check_grace_period
      description: Length of time in seconds after a new EC2 instance comes into service that Auto Scaling starts checking its health, if unspecified then the current group value will be used.
    - name: instance_type
      description: The type of the istance to launch in ASG, if unspecified then the current group value will be used.

## Dependencies ##

 - The AWS EC2 ASG is already provisioned

## Example Playbook For Full Deployment ##


```
---
- hosts: localhost
  connection: local
  vars:
    # set common_java_opts in supervisor using the ami baking playbook!
    app_memory_java_opts: >-
      -Xms1g -Xmx1g -XX:PermSize=512m -XX:MaxPermSize=512m
    instance_count: 1
  roles:
    - role: ansible-blue_green-lc-deploy
      aws_region: ap-southeast-1

      service_name: tsiasg
      cluster_role: app
      service_environment: production

      ami_id: ami-0a1b2c3d4e5f67890

      asg_name: tsiasg-app-abcdef0123456789
      asg_min_size: "{{ instance_count }}"
      asg_max_size: "{{ instance_count }}"
      asg_desired_capacity: "{{ instance_count }}"

      canary_release_count: 0

      canary_release_instance_ids: []

      # Set this to false if you have done canary release
      # for AMI version for optimal instance cost.
      # Setting this to true with canary release already done
      # will relaunch the new instances again with same AMI.
      replace_new_instances: false

      instance_user_data: |
        #cloud-config
        bootcmd:
        - echo "succeed"
        runcmd:
        - /opt/init/init-instance /dummy/data/dd.key
```

## Example Playbook For Canary Deployment ##


```
---
- hosts: localhost
  connection: local
  vars:
    # set common_java_opts in supervisor using the ami baking playbook!
    app_memory_java_opts: >-
      -Xms1g -Xmx1g -XX:PermSize=512m -XX:MaxPermSize=512m
    instance_count: 1
  roles:
    - role: ansible-blue_green-lc-deploy
      aws_region: ap-southeast-1

      service_name: tsiasg
      cluster_role: app
      service_environment: production

      ami_id: ami-0a1b2c3d4e5f67890

      asg_name: tsiasg-app-abcdef0123456789
      asg_min_size: "{{ instance_count }}"
      asg_max_size: "{{ instance_count }}"
      asg_desired_capacity: "{{ instance_count }}"

      # Canary release count should be less than ASG
      # instance size, otherwise deployment will fail.
      # If canary_release_count > 0 then canary_release_instance_ids should not be used.
      # Both variables cannot be used at same time
      canary_release_count: 3

      # Set this to instance ids to be replaced in canary release.
      # No of instances should be strictly less than instance count in ASG.
      # Incorrect instance ids will result in failed deployment.
      # If canary_release_instance_ids size > 0, canary_release_count should not be used.
      # Both variables cannot be used at same time
      canary_release_instance_ids: ["i-97593jjeief8488", "i-97593juoo98"]

      replace_new_instances: true

      instance_user_data: |
        #cloud-config
        bootcmd:
        - echo "succeed"
        runcmd:
        - /opt/init/init-instance /dummy/data/dd.key
```

## License

MIT

## Author Information ##

[Salvian Reynaldi](https://github.com/SalzzZ)

## Contributors ##

- [Rafi Kurnia Putra](https://github.com/rafikurnia)
