### Podman Ansible Updates

A very simple playbook to list podman containers, list and apply updates to them across various users. 

### Setup
- Setup host in inventory
- Adjust users in vars/users.yml

### Running

#### become vs ssh playbook:

On the host you can either use become (using the sudo password or root login) or ssh to switch between your users list.
- using ssh for each user, `ansible-playbook ssh.yml`.
- using ansible's `become` (sudo su -c), `ansible-playbook become.yml -K`.

#### Using tags

You can pass the playbook the tags which you want to run. 
Current tags:
- ps: runs `podman ps -a` on your users.
- checkupdates: runs `podman auto-update --dry-run` on your users.
- applyupdates: runs `podman auto-update` on your users.
Running `ansible-playbook {ssh/become}.yml` without any tags will run `ps` and `checkupdates`

Pass in multiple tags to run tasks at once:
- ansible-playbook ssh.yml --tags "checkupdates, applyupdates"


#### Passing only a subset of users.

You can use the normal ansible arguments as usual. For example to override the users.yml file and passing users on runtime you would pass `-e "users=[user1, user2]"`.

##### Sample output:

```
PLAY [Podman ansible updates] *************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************
ok: [host]

TASK [Querying podman for containers] ******************************************************************************************************************************************************************************************************
changed: [host] => (item=user1)
changed: [host] => (item=user2)
changed: [host] => (item=user3)

TASK [debug] *******************************************************************************************************************************************************************************************************************************
user1:
CONTAINER ID  IMAGE                                 COMMAND     CREATED         STATUS         PORTS                 NAMES
17cb9979aa65  docker.io/library/postgres:16.1       postgres    14 minutes ago  Up 14 minutes                        postgres

user2:
CONTAINER ID  IMAGE                                 COMMAND     CREATED         STATUS         PORTS                 NAMES
17cb9979aa65  docker.io/library/postgres:16.1       postgres    14 minutes ago  Up 14 minutes                        postgres

user3:
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES

TASK [Applying podman auto-update] *********************************************************************************************************************************************************************************************************
changed: [host] => (item=user1)
changed: [host] => (item=user2)
changed: [host] => (item=user3)

TASK [debug] *******************************************************************************************************************************************************************************************************************************
user1: 
            UNIT                            CONTAINER                    IMAGE                 POLICY      UPDATED
            postgres.service   17cb9979aa65 (postgres)   docker.io/library/postgres:16.1       registry    false

user2: 
            UNIT               CONTAINER                 IMAGE                                 POLICY      UPDATED
            postgres.service   17cb9979aa65 (postgres)   docker.io/library/postgres:16.1       registry    false

PLAY RECAP *********************************************************************************************************************************************************************************************************************************
host                     : ok=x    changed=y    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
