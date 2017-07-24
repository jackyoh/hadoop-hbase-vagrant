Test Command
```
ansible-playbook --private-key /home/user1/hadoop-hbase-vagrant/.vagrant/machines/default/virtualbox/private_key -i inventory playbook.yml
```

Variable
```
ansible-playbook --private-key /home/user1/hadoop-hbase-vagrant/.vagrant/machines/default/virtualbox/private_key -i inventory playbook.yml --extra-vars "key1=value1"
```
